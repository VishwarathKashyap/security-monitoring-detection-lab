# Incident 001 — SSH Brute Force

## Incident Summary

A controlled SSH brute-force simulation was performed against the
`familyserver-linux` endpoint to validate Wazuh's authentication monitoring
and brute-force detection capabilities.

The activity generated repeated SSH authentication failures and triggered
Wazuh rules 5710 and 5712.

---

## Classification

**Category:** Credential Access

**Attack Type:** SSH Brute Force / Password Guessing

**Environment:** Controlled Home Lab

**Status:** Detected — No Confirmed Compromise

---

## Target

| Field | Value |
|---|---|
| Host | familyserver-linux |
| Service | SSH |
| Port | 22 |
| Target User | nonexistentuser |
| Source IP | 127.0.0.1 |

---

## Attack Simulation

Repeated SSH authentication attempts were generated against a
non-existent account using incorrect passwords.

```text
Attack Simulation
       ↓
SSH Service
       ↓
Invalid User
       ↓
Repeated Authentication Failures
       ↓
Linux Journald
       ↓
Wazuh Agent
```

---

## Observed Telemetry

The endpoint generated authentication events including:

```text
Invalid user nonexistentuser
pam_unix(sshd:auth): authentication failure
Failed password for invalid user nonexistentuser
Connection closed by invalid user
```

---

## Wazuh Detection

### Rule 5710

**Description:**

```text
sshd: Attempt to login using a non-existent user
```

**Level:** 5

**MITRE ATT&CK:**

- T1110.001 — Password Guessing
- T1021.004 — SSH

---

### Rule 5712

**Description:**

```text
sshd: brute force trying to get access to the system.
Non existent user.
```

**Level:** 10

**Frequency:** 8

**MITRE ATT&CK:**

- T1110 — Brute Force

---

## Detection Timeline

```text
SSH Authentication Attempts
            ↓
Invalid User Detection
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

## Investigation

### Who?

**Source IP:**

```text
127.0.0.1
```

### What?

Repeated failed SSH authentication attempts against a non-existent user.

### When?

The activity occurred within a short authentication window and reached
the threshold required for Wazuh's brute-force detection rule.

### Where?

**Target endpoint:**

```text
familyserver-linux
```

### How?

The simulation repeatedly attempted password authentication against SSH.

### Impact

No successful authentication was observed.

No evidence of account compromise was identified.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Brute Force | T1110 | Repeated authentication attempts |
| Password Guessing | T1110.001 | Password guessing activity |
| SSH | T1021.004 | SSH remote service |

---

## Response

No containment action was performed because the activity was intentionally
generated as a controlled laboratory simulation.

In a real incident, the response would depend on additional evidence such as
successful authentication, source reputation, account compromise, or
post-authentication activity.

---

## Evidence

Evidence collected:

- Linux authentication logs
- Wazuh Rule 5710 events
- Wazuh Rule 5712 alert
- MITRE ATT&CK mapping
- Wazuh dashboard investigation
- Incident analysis

Screenshots are stored in the `screenshots/` directory.

---

## Lessons Learned

1. Individual authentication failures can provide useful security telemetry.
2. Wazuh can correlate repeated events into a higher-level brute-force alert.
3. A security alert must be investigated before determining whether a
   compromise actually occurred.
4. MITRE ATT&CK provides a standardized way to classify attacker behavior.

---

## Conclusion

The SSH brute-force scenario successfully demonstrated the complete workflow:

```text
Attack
  ↓
Telemetry
  ↓
Detection
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

## Dashboard Evidence

![Wazuh SSH Brute Force Detection](screenshots/ssh-bruteforce-rule.png)
