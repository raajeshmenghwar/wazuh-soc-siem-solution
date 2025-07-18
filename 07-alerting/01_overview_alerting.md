# Alerting in Wazuh

Wazuh provides a powerful and extensible alerting framework that enables organizations to receive real-time notifications of security events, anomalies, and other important system events. These alerts help security analysts take immediate action and improve incident response.

---

## Types of Integrations

Wazuh supports multiple third-party integrations for alerting and automated responses. Here are some commonly used alerting options:

* [Email Integration](02_email_integration.md): Send alerts directly to your organization’s inbox.
* [Slack Integration](03_slack_integration.md): Send alerts to specific Slack channels to notify teams.
* [Telegram Bot Integration](04_telegram_integration.md): Receive alerts via a Telegram bot for real-time mobile updates.
* Webhook Integrations: For pushing alerts to custom web services.
* SIEM Tools: Integration with tools like Splunk, Elastic Stack.
* Ticketing Systems: Integration with platforms like TheHive and ServiceNow.

---

## Why Integrate Alerting?

Effective alerting ensures:

* Real-time detection of critical threats.
* Faster incident response.
* Centralized monitoring.
* Reduced manual log reviews.

Wazuh generates alerts based on rules defined in its ruleset. These alerts are customizable and can be filtered, tagged, or sent to specific external systems through integrations.

---

## How Alerts Work

1. **Event Detection**: Logs from agents are analyzed.
2. **Rule Match**: If a rule matches, an alert is generated.
3. **Alert Handling**: Alert is formatted and sent to the appropriate output (e.g., email, Slack).

##  Alert Lifecycle

- **Log Collection** – Wazuh agents collect logs from endpoints.
- **Decoding** – Logs are normalized using decoders.
- **Rule Matching** – Rules are applied to decoded logs.
- **Alert Generation** – If a rule matches, an alert is generated.
- **Notification Sent** – Based on configuration, alerts are forwarded.

##  Alert Components

| Component       | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| Rule ID         | The ID associated with the triggered rule.                                 |
| Alert Level     | A severity level from 0 (informational) to 15 (critical).                   |
| Source Log      | The raw log data that caused the alert.                                    |
| Rule Description| The human-readable message explaining the alert.                           |
| Integration     | Method by which the alert is forwarded (e.g., email, Slack, webhook, etc). |


Refer to individual integration files for step-by-step instructions on each integration type.

---

