# Alerting System Overview in Wazuh

Wazuh is a powerful open-source Security Information and Event Management (SIEM) tool. One of its core capabilities is **alerting**, which informs administrators of security events that require attention.

This document provides an overview of how Wazuh's alerting system works, its components, and what options are available for customization.

---

##  What is an Alert?

An **alert** is a notification generated when Wazuh detects an event that matches a specific rule defined in its configuration.

---

##  Alert Components

| Component       | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| Rule ID         | The ID associated with the triggered rule.                                 |
| Alert Level     | A severity level from 0 (informational) to 15 (critical).                   |
| Source Log      | The raw log data that caused the alert.                                    |
| Rule Description| The human-readable message explaining the alert.                           |
| Integration     | Method by which the alert is forwarded (e.g., email, Slack, webhook, etc). |

---

##  Alert Lifecycle

1. **Log Collection** – Wazuh agents collect logs from endpoints.
2. **Decoding** – Logs are normalized using decoders.
3. **Rule Matching** – Rules are applied to decoded logs.
4. **Alert Generation** – If a rule matches, an alert is generated.
5. **Notification Sent** – Based on configuration, alerts are forwarded.

---

##  Important Files

| File/Directory                   | Purpose                                |
|----------------------------------|----------------------------------------|
| `/var/ossec/etc/rules`           | Custom and pre-defined rules.          |
| `/var/ossec/logs/alerts/alerts.json` | Raw alerts in JSON format.            |
| `/var/ossec/etc/ossec.conf`      | Core configuration, including alerting.|

---

##  Alerting Use Cases

- Unauthorized access attempts
- Malware execution detection
- File integrity changes
- Network port scanning

---

##  Next Steps

- Configure built-in integrations like Email.
- Create custom alerting mechanisms.
```

---
