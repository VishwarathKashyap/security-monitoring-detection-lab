# Security Monitoring & Detection Lab

A hands-on cybersecurity home lab focused on security monitoring, SIEM operations, threat detection, alert triage, incident investigation, and MITRE ATT&CK mapping using Wazuh.

## Project Overview

This project is a self-hosted Security Operations Center (SOC) lab deployed on a home server.

The lab is designed to simulate a real security monitoring workflow:

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
Investigation
       ↓
MITRE ATT&CK Mapping
       ↓
Incident Documentation
