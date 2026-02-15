# SOC Home Lab – Wazuh Deployment

## Overview

This repository documents the design, deployment, and refinement of a SOC-focused home lab environment.

The lab is designed to develop practical skills in:

- Security monitoring  
- Threat detection  
- Log analysis  
- Network visibility  
- Incident investigation workflows  

Rather than pursuing unnecessary complexity, the environment prioritizes clarity, segmentation, and observability.

---

## Objectives

This lab was built to:

- Simulate realistic security monitoring scenarios  
- Centralize logs across infrastructure components  
- Provide visibility into host and network activity  
- Support detection engineering experimentation  
- Reinforce structured troubleshooting workflows  

---

## Environment Architecture

Core components of the lab:

- **Firewall / Network Edge:** pfSense  
- **Network Visibility / IDS:** Suricata  
- **Compute Platform:** ProxMox  
- **Security Platform:** Wazuh  
- **Log Sources:**  
  - Firewall logs  
  - IDS alerts  
  - Linux system events  
  - Endpoint telemetry  

---

## Key Components

### pfSense

Acts as the network perimeter and traffic control point.

Responsibilities include:

- Network segmentation  
- Firewall policy enforcement  
- DNS resolution and host overrides  
- Syslog forwarding  

---

### Suricata

Provides network-level detection and inspection.

Responsibilities include:

- Traffic inspection  
- Signature-based detection  
- Alert generation  

---

### ProxMox Virtual Enviornment 9.1.1

Serves as the virtualization platform hosting lab workloads.

Responsibilities include:

- Virtual machine management  
- Storage experimentation  
- Isolation of vulnerable workloads  

---

### Wazuh

Functions as the centralized security monitoring and analysis platform.

Responsibilities include:

- Log aggregation  
- Event correlation  
- Agent-based telemetry  
- Detection and alerting  

---

## TLS / Certificate Implementation

HTTPS/TLS was implemented to mirror production-like security controls.

Key elements:

- Let’s Encrypt certificates  
- DNS-01 validation via Cloudflare  
- Internal DNS resolution using pfSense host overrides  
- LAN-only access design  

This ensures:

- Encrypted communications  
- Proper hostname validation  
- Elimination of self-signed certificate warnings  

---

## Certificate Issuance Workflow

Certificates are issued using the pfSense ACME package with DNS validation.

### Example Configuration Steps

### 1. Cloudflare API Token

A Cloudflare API token was created with the following permissions:

- Zone → DNS → Edit  
- Zone → Zone → Read  

---

### 2. ACME Account Key Creation

pfSense → Services → ACME → Account Keys

```bash
Generate New Account Key
Register ACME Account Key
```

---

### 3. Certificate Creation

pfSense → Services → ACME → Certificates

Subject Alternative Names (SAN):

```bash
techysec.com
*.techysec.com
```

Validation Method:

```bash
DNS-Cloudflare
```

---

### 4. DNS Validation Process

During issuance, ACME performs DNS challenges:

```bash
_acme-challenge.techysec.com
```

TXT records are automatically created and removed.

---

## Security Design Considerations

This lab intentionally incorporates:

- Network segmentation  
- Principle of least privilege  
- Encrypted management interfaces  
- Controlled service exposure  
- Realistic failure and troubleshooting scenarios  

---

## Lessons Learned

Key practical concepts reinforced:

- Certificate trust vs encryption  
- DNS vs IP-based access implications  
- TLS hostname validation  
- Agent-manager compatibility  
- Service binding vs accessibility  
- Structured troubleshooting methodology  

---

## Purpose of This Repository

This repository serves as:

- A technical knowledge base  
- A troubleshooting reference  
- A demonstration of SOC / detection skill development  
- A living document of iterative lab refinement  

---

## Future Enhancements

Planned improvements:

- Reverse proxy implementation  
- Additional log source integrations
- Custom detection rule development  
- Incident simulation scenarios  
- Expanded endpoint coverage  
