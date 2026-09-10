# Security Monitoring & Detection Lab

A self-hosted cybersecurity lab built to practice **SIEM, security monitoring,
endpoint telemetry, threat detection, alert triage, incident investigation,
vulnerability detection, and MITRE ATT&CK mapping** using Wazuh.

The lab is hosted on a personal Ubuntu home server and uses controlled
security scenarios to simulate SOC workflows in an authorized environment.

---

## Overview

The goal of this project is to understand how a SOC analyst monitors endpoints,
detects suspicious activity, investigates alerts, and documents security
incidents.

The project follows this workflow:

```text
Endpoint Activity
       ↓
Wazuh Agent
       ↓
Wazuh Manager
       ↓
Detection Rules
       ↓
Security Alert
       ↓
Alert Triage
       ↓
Investigation
       ↓
MITRE ATT&CK Mapping
       ↓
Incident Documentation
```

---

## Objectives

- Build and maintain a functional SIEM environment
- Collect endpoint security telemetry
- Understand Linux authentication and system events
- Detect malicious and suspicious activity
- Perform alert triage and incident investigation
- Map detected activity to MITRE ATT&CK
- Develop custom detection rules
- Practice incident response workflows
- Document security incidents and evidence

---

## Architecture

```text
                         HOME NETWORK
                              │
                              ▼
                    ┌──────────────────┐
                    │   Ubuntu Server  │
                    │   familyserver   │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
       ┌──────────────┐ ┌──────────────┐ ┌───────────────┐
       │ Wazuh        │ │ Wazuh        │ │ Wazuh         │
       │ Manager      │ │ Indexer      │ │ Dashboard     │
       └──────┬───────┘ └──────────────┘ └───────────────┘
              │
              │ Security Events
              ▼
       ┌──────────────────┐
       │   Wazuh Agent    │
       │ familyserver-    │
       │ linux            │
       └────────┬─────────┘
                │
        ┌───────┼───────────┐
        │       │           │
        ▼       ▼           ▼
   System Logs  Security   System
                Events     Inventory
```

### Wazuh Docker Network

The Wazuh components run inside a dedicated Docker bridge network,
isolated from the existing home-server application networks.

```text
Wazuh Docker Network
       │
       ├── Wazuh Manager
       │    └── 172.21.0.2
       │
       ├── Wazuh Dashboard
       │    └── 172.21.0.3
       │
       └── Wazuh Indexer
            └── 172.21.0.4
```

The Wazuh stack is isolated from the existing Docker networks used by
Immich, Jellyfin, and Open WebUI.

---

## Technologies Used

### SIEM & Monitoring

- Wazuh
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Docker
- Docker Compose

### Endpoint

- Ubuntu Linux
- Wazuh Agent
- systemd/journald
- OpenSSH

### Security Concepts

- SIEM
- Security Monitoring
- Log Analysis
- Alert Triage
- Threat Detection
- Incident Investigation
- Vulnerability Detection
- MITRE ATT&CK
- Indicators of Compromise (IoCs)
- TTP Analysis
- Incident Response

---

## Lab Environment

| Component | Details |
|---|---|
| Host OS | Ubuntu 26.04.1 LTS |
| CPU | Intel Core i5-10300H |
| CPU Cores | 4 Cores / 8 Threads |
| RAM | 14 GB |
| Docker | 29.1.3 |
| Docker Compose | 2.40.3 |
| Wazuh | 4.14.7 |
| Agent | familyserver-linux |
| Architecture | amd64 |

---

## Deployment

The Wazuh stack was deployed using the official Wazuh Docker deployment.

### Core Components

```text
Wazuh Manager
Wazuh Indexer
Wazuh Dashboard
```

The components communicate through a dedicated Docker network.

### Dashboard Access

To avoid conflicts with existing services on the server, the Wazuh
Dashboard container port `5601` is mapped to host port `8443`.

```text
Host Port:      8443
Container Port: 5601
Protocol:       HTTPS
```

The dashboard is accessible from the local network through:

```text
https://<server-ip>:8443
```

### Linux Endpoint Enrollment

The Ubuntu server was enrolled as a Wazuh agent for endpoint monitoring.

```text
Agent ID:   002
Agent Name: familyserver-linux
Status:     Active
```

The agent collects endpoint telemetry and forwards security events to the
Wazuh Manager for analysis and detection.

---

## Data Flow

```text
Linux Endpoint
      │
      ▼
Wazuh Agent
      │
      ▼
Wazuh Manager
      │
      ├── Log Analysis
      ├── Detection Rules
      └── Vulnerability Detection
      │
      ▼
Wazuh Indexer
      │
      ▼
Wazuh Dashboard
      │
      ▼
Alert Investigation
```

---

## Detection Scenarios

### Scenario 01 — SSH Brute Force

A controlled SSH brute-force simulation was performed against the
`familyserver-linux` endpoint.

```text
Repeated SSH Attempts
          ↓
Linux Authentication Logs
          ↓
Wazuh Agent
          ↓
Wazuh Manager
          ↓
Detection Rule
          ↓
Security Alert
          ↓
SOC Investigation
```

### Attack Simulation

A nonexistent SSH account was repeatedly used with incorrect passwords
against the local SSH service.

```text
Controlled Attack
       ↓
SSH
       ↓
Invalid User
       ↓
Repeated Authentication Failures
```

### Observed Telemetry

The Linux system generated events including:

```text
Invalid user nonexistentuser
pam_unix(sshd:auth): authentication failure
Failed password for invalid user nonexistentuser
Connection closed by invalid user
```

### Wazuh Detection

#### Rule 5710

```text
Description:
sshd: Attempt to login using a non-existent user

Level:
5
```

MITRE ATT&CK mappings:

```text
T1110.001 - Password Guessing
T1021.004 - SSH
```

#### Rule 5712

```text
Description:
sshd: brute force trying to get access to the system.
Non existent user.

Level:
10

Frequency:
8
```

MITRE ATT&CK:

```text
T1110 - Brute Force
```

### Detection Flow

```text
SSH Authentication Failure
            ↓
Linux journald
            ↓
Wazuh Agent
            ↓
SSH Decoder
            ↓
Rule 5710
            ↓
Repeated Authentication Failures
            ↓
Rule 5712
            ↓
Level 10 Alert
            ↓
SOC Investigation
```

---

## Incident Investigation

The alert was investigated using Wazuh security events and endpoint
authentication telemetry.

### Investigation Framework

```text
WHO?
WHAT?
WHEN?
WHERE?
HOW?
IMPACT?
RESPONSE?
```

### Incident Findings

| Field | Finding |
|---|---|
| Source IP | 127.0.0.1 |
| Target | familyserver-linux |
| Service | SSH |
| Target User | nonexistentuser |
| Detection Rule | 5712 |
| Severity | Level 10 |
| MITRE ATT&CK | T1110 - Brute Force |
| Compromise | No successful authentication observed |

### Initial Assessment

The activity represented a controlled SSH brute-force and password-guessing
simulation performed against the lab endpoint.

No successful authentication or account compromise was observed.

### Response

No containment action was performed because the activity was intentionally
generated as part of a controlled security test.

---

## MITRE ATT&CK Mapping

| Technique | ID | Relevance |
|---|---|---|
| Brute Force | T1110 | Repeated authentication attempts |
| Password Guessing | T1110.001 | Incorrect password attempts |
| SSH | T1021.004 | SSH remote service |

---

## Evidence

Evidence collected during investigations includes:

- Linux authentication logs
- Wazuh security alerts
- Detection rule information
- MITRE ATT&CK mappings
- Incident investigation notes
- Dashboard screenshots

Evidence will be organized by detection scenario as the project expands.

---

## Incident Response Workflow

The project follows a structured SOC investigation workflow:

```text
Alert
  ↓
Validation
  ↓
Triage
  ↓
Investigation
  ↓
Classification
  ↓
Impact Assessment
  ↓
Containment
  ↓
Remediation
  ↓
Documentation
```

---

## Project Roadmap

### Completed

- [x] Wazuh Manager deployment
- [x] Wazuh Indexer deployment
- [x] Wazuh Dashboard deployment
- [x] Dedicated Docker network
- [x] Linux Wazuh Agent enrollment
- [x] Endpoint telemetry collection
- [x] Vulnerability detection
- [x] SSH brute-force simulation
- [x] Rule 5710 validation
- [x] Rule 5712 validation
- [x] MITRE ATT&CK mapping
- [x] Initial incident investigation

### Planned

- [ ] Suspicious authentication detection
- [ ] Privilege escalation detection
- [ ] Suspicious process detection
- [ ] File integrity monitoring
- [ ] Persistence detection
- [ ] Windows endpoint monitoring
- [ ] Sysmon integration
- [ ] PowerShell detection
- [ ] Custom Wazuh detection rules
- [ ] Detection tuning
- [ ] False-positive analysis
- [ ] Threat hunting
- [ ] Automated response
- [ ] Python-based security automation

---

## Key Learnings

### Event vs Alert

An **event** is observed system activity.

An **alert** is activity that matches a security detection rule and is
considered security-relevant.

```text
Event
  ↓
Detection Rule
  ↓
Alert
```

### Alert vs Compromise

A security alert does not automatically mean that a system has been
compromised.

The analyst must investigate the available evidence and determine the
actual impact.

```text
Alert
  ↓
Validate
  ↓
Investigate
  ↓
Determine Impact
  ↓
Respond
```

### SOC Investigation

The investigation process focuses on understanding:

```text
Who?
What?
When?
Where?
How?
Impact?
Response?
```

---

## Project Status

**Status:** Active Development

The initial SIEM deployment, Linux endpoint monitoring, vulnerability
detection, and SSH brute-force detection scenario have been completed.

The next stage focuses on custom detection engineering, additional attack
scenarios, threat hunting, and incident response automation.

---

## Disclaimer

This project is performed in a controlled home-lab environment using
systems and services owned and administered by me.

All attack simulations are conducted only against authorized laboratory
systems.

---

## Author

**Vishwarath Kashyap**

[GitHub](https://github.com/VishwarathKashyap)
