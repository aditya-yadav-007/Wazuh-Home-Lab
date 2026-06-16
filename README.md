# Wazuh Home Security Lab

A self-hosted SIEM lab demonstrating end-to-end attack detection — from exploitation of a vulnerable web app to real-time alert generation and incident response.

## Architecture

```
Attacker (Host) ---> OWASP Juice Shop (VM, vulnerable target)
                              |
                        HTTP/Network traffic
                              |
                       Wazuh Agent (VM)
                              |
                       Wazuh Manager (VM)
                              |
                    Wazuh Dashboard (alerts/logs)
```

- **Hypervisor:** VirtualBox/VMware
- **OS:** Ubuntu Server 22.04 LTS
- **SIEM:** Wazuh 4.x (Manager + Indexer + Dashboard)
- **Target:** OWASP Juice Shop (Docker container)
- **Resources:** 4GB+ RAM, 2 vCPU

## Setup

1. Deploy Wazuh single-node stack on Ubuntu VM (`wazuh-install.sh`).
2. Run OWASP Juice Shop in Docker on the same network segment:
   ```bash
   docker run -d -p 3000:3000 bkimminich/juice-shop
   ```
3. Install Wazuh agent on the Juice Shop host, register with manager.
4. Configure log collection for HTTP access logs and system events.

## Detection Rules Used

| Rule ID | Description | Trigger |
|---|---|---|
| 31103 | SQL Injection attempt detected | Malicious SQL patterns in HTTP requests |
| 31106 | XSS attempt detected | Script injection patterns in request params |

## Attack Simulation

Exploited known Juice Shop challenges to generate real alert traffic:

- **SQL Injection:** Login bypass via `' OR 1=1--` on `/rest/user/login`
- **XSS:** Reflected XSS via search/query parameters
- **Broken Auth:** Forced browsing to admin endpoints

## Incident Response Workflow

1. **Detect:** Wazuh dashboard fires alert (rule 31103/31106) on malicious request.
2. **Triage:** Cross-reference alert timestamp with raw HTTP log and source IP.
3. **Analyze:** Confirm payload signature, map to OWASP Top 10 category.
4. **Document:** Record finding — vulnerability, evidence, severity, remediation.
5. **Remediate:** Recommend input validation / parameterized queries / output encoding.

## Sample Finding (Documented)

**Finding:** SQL Injection — Login Bypass
**Endpoint:** `POST /rest/user/login`
**Payload:** `email=' OR 1=1--&password=x`
**Wazuh Alert:** Rule 31103, Level 10
**Evidence:** Successful auth bypass returning valid JWT despite invalid credentials
**Remediation:** Use parameterized queries; validate/sanitize all user input server-side

## Files in This Repo

```
/screenshots      - Dashboard alerts, rule triggers
/logs             - Sample raw HTTP logs + Wazuh alert JSON
/configs          - ossec.conf snippets, agent config
incident-report.pdf - Full writeup of attack chain
```

## Skills Demonstrated

SIEM deployment & tuning, log analysis, OWASP Top 10 exploitation, detection engineering, incident documentation.