# Incident 002 — Sudo Privilege Escalation Detection

## Incident Summary

A controlled privilege-escalation test was performed on the
`familyserver-linux` endpoint to validate Wazuh's ability to detect
successful sudo execution to the `root` user.

The test executed the `/usr/bin/id` command with elevated privileges,
resulting in a Wazuh Level 3 alert under Rule 5402.

---

## Classification

**Category:** Privilege Escalation

**Activity Type:** Sudo to Root

**Environment:** Controlled Home Lab

**Status:** Detected — Authorized Activity

---

## Target

| Field | Value |
|---|---|
| Host | familyserver-linux |
| Source User | vishwa |
| Target User | root |
| Command | `/usr/bin/id` |
| Detection Rule | 5402 |
| Severity | Level 3 |

---

## Activity Simulation

A controlled privileged command was executed using `sudo`:

```bash
sudo id
```

The command returned the identity of the effective user and executed with
root privileges.

```text
Normal User
     ↓
sudo
     ↓
Root Privilege
     ↓
/usr/bin/id
     ↓
Wazuh Detection
```

---

## Wazuh Detection

### Rule 5402

**Description:**

```text
Successful sudo to ROOT executed.
```

**Level:** 3

**Rule File:**

```text
0020-syslog_rules.xml
```

**Groups:**

```text
syslog
sudo
```

### Detection Logic

Rule 5402 is associated with sudo activity resulting in execution as
`USER=root` with a command.

The observed event contained:

```text
Source User: vishwa
Destination User: root
Command: /usr/bin/id
```

### MITRE ATT&CK

The Wazuh rule associates the activity with:

```text
Sudo and Sudo Caching
```

MITRE ATT&CK tactics shown by the Wazuh rule:

```text
Privilege Escalation
Defense Evasion
```

---

## Detection Flow

```text
User Activity
      ↓
sudo
      ↓
USER=root
      ↓
/usr/bin/id
      ↓
Wazuh Agent
      ↓
Wazuh Manager
      ↓
Rule 5402
      ↓
Level 3 Alert
      ↓
SOC Investigation
```

---

## Investigation

### Who?

The command was executed by:

```text
vishwa
```

### What?

A successful sudo operation executed the `/usr/bin/id` command as
`root`.

### Where?

The activity occurred on:

```text
familyserver-linux
```

### When?

The event was recorded by Wazuh during the controlled test and is visible
in the Wazuh Threat Hunting event view.

### How?

The user explicitly invoked `sudo` to execute the `id` command with
elevated privileges.

### Impact

The command executed successfully with root privileges.

No unauthorized system modification or malicious activity was observed.

---

## Assessment

The detection was **valid**, but the activity was **authorized**.

This scenario demonstrates an important SOC principle:

```text
Detection
    ≠
Malicious Activity
```

A privileged execution event requires contextual investigation to determine
whether the activity was expected, suspicious, or malicious.

In this case:

```text
Detection:          Confirmed
Privilege Elevation: Confirmed
Authorized Activity: Confirmed
Compromise:         Not observed
Malicious Intent:   Not observed
```

---

## Response

No containment or remediation action was required because the privileged
command was intentionally executed as part of a controlled security test.

In a real environment, an unexpected sudo-to-root event would require
investigation of the user, command, timing, authorization, and surrounding
endpoint activity.

---

## MITRE ATT&CK

| Technique | Relevance |
|---|---|
| Sudo and Sudo Caching | Privileged command execution |
| Privilege Escalation | User transitioned to root privileges |
| Defense Evasion | Privileged execution can potentially support actions intended to evade controls |

---

## Evidence

Evidence collected during the investigation:

- Wazuh Rule 5402 alert
- Source user: `vishwa`
- Target user: `root`
- Command: `/usr/bin/id`
- Alert severity: Level 3
- Wazuh rule definition
- Wazuh Threat Hunting event
- Dashboard screenshot

### Dashboard Evidence

![Wazuh Sudo Privilege Escalation Detection](screenshots/privilege-escalation-5402.png)

---

## Lessons Learned

1. Wazuh can detect successful sudo execution to the root account.
2. Alert investigation requires identifying the source user, target user,
   command, and surrounding context.
3. A privilege-escalation alert does not automatically indicate malicious
   activity.
4. Security analysts must distinguish authorized administrative activity
   from suspicious privilege escalation.
5. Detection context is essential for accurate incident classification.

---

## Conclusion

The controlled sudo test successfully demonstrated endpoint monitoring,
privilege-escalation detection, alert triage, and contextual investigation.

The event was correctly detected by Wazuh Rule 5402 and determined to be
authorized administrative activity with no evidence of compromise.

```text
Privileged Activity
        ↓
Detection
        ↓
Alert Triage
        ↓
Contextual Investigation
        ↓
Authorized Activity
        ↓
No Containment Required
```
