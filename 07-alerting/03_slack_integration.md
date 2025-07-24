# Slack Integration with Wazuh Alerts

## Overview

[Slack](https://slack.com/) is a widely-used collaboration and messaging platform that enables teams to communicate in real time. Integrating Slack with Wazuh allows system administrators and security teams to receive immediate alerts from the Wazuh Security Information and Event Management (SIEM) system directly into their Slack channels. This ensures that critical security events are not missed and can be acted upon swiftly.

In this guide, we will walk through the process of integrating Wazuh alerts with Slack using a custom alerting script. We will also cover how to temporarily disable the integration when required.

---

## Why Integrate Wazuh with Slack?

- **Real-time visibility**: Receive alerts immediately as they are triggered by Wazuh rules.
- **Improved response time**: Take action faster by noticing high-severity alerts directly in Slack.
- **Centralized communication**: Bring your incident notifications into the same platform where teams already communicate.

---

## Prerequisites

- A working Wazuh manager setup.
- Access to the server running the Wazuh manager.
- Administrative access to a Slack workspace to create and manage incoming webhooks.
- Basic command-line knowledge of Linux.

---

## Step 1: Create an Incoming Webhook in Slack

1. Go to [Slack API: Incoming Webhooks](https://api.slack.com/messaging/webhooks).
2. Click **Create a Slack App**.
3. Choose **From scratch**, enter a name (e.g., `Wazuh Alerts`), and select your workspace.
4. Under **Add features and functionality**, select **Incoming Webhooks**.
5. Turn on **Activate Incoming Webhooks**.
6. Click **Add New Webhook to Workspace**.
7. Select the Slack channel where you want to receive alerts.
8. Copy the generated **Webhook URL**. You will use this in your script.

---

## Step 2: Create a Custom Slack Alert Script

Create a script in the Wazuh active response directory to handle Slack notifications.

```bash
sudo nano /var/ossec/active-response/bin/slack-alert.sh
````

Paste the following script:

```bash
#!/bin/bash

# Slack Webhook URL
WEBHOOK_URL="https://hooks.slack.com/services/XXXXX/XXXXX/XXXXXXXX"

# Read the alert JSON from stdin
ALERT=$(</dev/stdin)

# Extracting data using basic shell tools
ALERT_LEVEL=$(echo "$ALERT" | grep '"rule":' -A 10 | grep '"level":' | awk -F': ' '{print $2}' | sed 's/,//')
DESCRIPTION=$(echo "$ALERT" | grep '"description":' | head -1 | cut -d ':' -f2- | sed 's/[",]//g' | xargs)
LOG=$(echo "$ALERT" | grep '"full_log":' | cut -d ':' -f2- | sed 's/[",]//g' | xargs)

# Compose the Slack message payload
PAYLOAD="{
  \"text\": \"*Wazuh Alert (Level $ALERT_LEVEL)*\n*Description:* $DESCRIPTION\n*Log:* $LOG\"
}"

# Send the payload to Slack
curl -X POST -H 'Content-type: application/json' --data "$PAYLOAD" "$WEBHOOK_URL"
```

> **Note:** Replace the `WEBHOOK_URL` with your actual Slack webhook URL.

---

## Step 3: Make the Script Executable

```bash
sudo chmod +x /var/ossec/active-response/bin/slack-alert.sh
```

---

## Step 4: Register the Script in `ossec.conf`

Edit the Wazuh configuration to register the custom script and define when it should be executed.

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add the following under the `<command>` section:

```xml
<command>
  <name>slack-alert</name>
  <executable>slack-alert.sh</executable>
  <timeout_allowed>no</timeout_allowed>
</command>
```

Add the following under the `<active-response>` section (customize rule ID or level as per your requirement):

```xml
<active-response>
  <command>slack-alert</command>
  <location>all</location>
  <rules_id>1002</rules_id> <!-- Replace with appropriate rule ID -->
</active-response>
```

> You can find rule IDs in `/var/ossec/ruleset/rules/` or the logs Wazuh generates.

---

## Step 5: Restart Wazuh Manager

To apply the new configuration:

```bash
sudo systemctl restart wazuh-manager
```

---

## Step 6: Verify the Integration

Trigger a rule manually (e.g., failed SSH login), and observe the Slack channel for incoming alerts. The alert message will include the level, description, and full log line associated with the incident.

---

## Understanding What Happens Behind the Scenes

1. Wazuh detects a log/event that matches a specific rule (e.g., failed login).
2. If that rule is associated with an `<active-response>` entry, it runs the corresponding script.
3. The `slack-alert.sh` script receives the alert data (in JSON) via stdin.
4. The script extracts key information (level, description, log).
5. It formats the alert into a readable Slack message.
6. Finally, it sends the message to the Slack channel using the webhook URL.

---

## Optional: Temporarily Disabling Slack Alerts

Sometimes, you may want to stop sending alerts while making system changes.

### Method: Rename the Script Temporarily

```bash
sudo mv /var/ossec/active-response/bin/slack-alert.sh /var/ossec/active-response/bin/slack-alert.sh.disabled
```

To re-enable:

```bash
sudo mv /var/ossec/active-response/bin/slack-alert.sh.disabled /var/ossec/active-response/bin/slack-alert.sh
```

This approach ensures Slack alerts are paused without altering Wazuh configuration files.

---

## Conclusion

Integrating Slack with Wazuh provides instant visibility into potential security threats by delivering alerts directly to your team’s communication channel. By using a simple, maintainable bash script, administrators can keep their incident response workflow tightly integrated and highly responsive.

For production setups, it's advisable to further harden the script with input sanitization, log retention, and notification throttling mechanisms.

---

## References

* [Wazuh Documentation](https://documentation.wazuh.com/)
* [Slack Incoming Webhooks](https://api.slack.com/messaging/webhooks)

