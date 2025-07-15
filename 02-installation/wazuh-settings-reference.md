# Wazuh Settings Reference for Beginners

This guide introduces essential configuration files, service commands, and troubleshooting steps for managing a Wazuh SIEM setup. 

---

## Core Configuration Files

### 1. `ossec.conf`

- **Purpose:** Main configuration file for Wazuh Manager and Agents.
- **Location:** `/var/ossec/etc/ossec.conf`
- **What it Does:** Controls nearly everything: log collection, rules to load, remote agent communication, alert settings, email notifications, active responses, and more.

#### Key Concepts

- Written in **XML** format.
- Each section is enclosed in tags like `<global>`, `<rules>`, `<remote>`.
- Changes to this file **require restarting the Wazuh Manager**.

#### Sample Block

```xml
<ossec_config>
  <global>
    <email_notification>yes</email_notification>
    <email_to>admin@example.com</email_to>
  </global>

  <rules>
    <include>rules/local_rules.xml</include>
  </rules>

  <remote>
    <connection>secure</connection>
  </remote>
</ossec_config>
````

#### Common Additional Blocks

```xml
<active-response>
  <command>disable-account</command>
  <location>local</location>
  <rules_id>100201</rules_id>
</active-response>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>
```

| Tag                 | Purpose                                        |
| ------------------- | ---------------------------------------------- |
| `<global>`          | General options like email alerts and timeouts |
| `<rules>`           | Rule files to include                          |
| `<remote>`          | Agent-manager communication                    |
| `<active-response>` | Define automatic actions triggered by alerts   |
| `<localfile>`       | Add log files you want Wazuh to monitor        |

#### Validation Tip

Run this command to validate XML syntax:

```bash
xmllint /var/ossec/etc/ossec.conf --noout
```

---

### 2. `ruleset/rules/`
- **Purpose:** Directory containing default and custom detection rules.
- **Location:** `/var/ossec/ruleset/rules/`
- **What it does:** Houses `.xml` files that define how Wazuh detects threats.

### 3. `agent.conf`
- **Purpose:** Centralized configuration for agents, managed by the manager.
- **Location:** `/var/ossec/etc/shared/agent.conf`
- **What it does:** Pushes settings to agents based on their group.

### 4. `local_rules.xml`
- **Purpose:** File for custom rules.
- **Location:** `/var/ossec/etc/rules/local_rules.xml`
- **What it does:** Lets you add or override detection rules without editing core files.

### 5. `decoders/`
- **Purpose:** Directory for log decoders.
- **Location:** `/var/ossec/etc/decoders/`
- **What it does:** Contains `.xml` files that parse and normalize incoming logs.

### 6. `wazuh-cluster.json` (if using clustering)
- **Purpose:** Cluster configuration file.
- **Location:** `/var/ossec/etc/wazuh-cluster.json`
- **What it does:** Defines cluster nodes, roles, and communication settings.

---

## Wazuh Manager Service Commands

| Task    | Command                                | Description                      |
| ------- | -------------------------------------- | -------------------------------- |
| Start   | `sudo systemctl start wazuh-manager`   | Starts the Wazuh Manager service |
| Stop    | `sudo systemctl stop wazuh-manager`    | Stops the Wazuh Manager          |
| Restart | `sudo systemctl restart wazuh-manager` | Reloads config and rules         |
| Status  | `sudo systemctl status wazuh-manager`  | Check if it's running            |
| Logs    | `sudo journalctl -u wazuh-manager`     | View service logs for issues     |

---

## Useful Paths Overview

| Item                | Path                                   |
| ------------------- | -------------------------------------- |
| Main Config         | `/var/ossec/etc/ossec.conf`            |
| Default Rules       | `/var/ossec/ruleset/rules/`            |
| Custom Rules        | `/var/ossec/etc/rules/local_rules.xml` |
| Decoders            | `/var/ossec/etc/decoders/`             |
| Shared Agent Config | `/var/ossec/etc/shared/agent.conf`     |
| Manager Logs        | `/var/ossec/logs/ossec.log`            |
| Cluster Config      | `/var/ossec/etc/wazuh-cluster.json`    |

---

## Why Restart After Changes?

Wazuh loads rule files, decoders, and `ossec.conf` only at startup. If you make a change and don’t restart, **your edits won’t take effect**.

---

## Troubleshooting Tips

| Problem                    | Solution                                          |
| -------------------------- | ------------------------------------------------- |
| Wazuh won't start          | Check XML syntax using `xmllint`                  |
| No logs in dashboard       | Ensure agents are connected and sending logs      |
| Config errors              | Use `journalctl -u wazuh-manager` for details     |
| Custom rule not triggering | Check alert level and if the rule is included     |
| Invalid file path          | Confirm `<include>` and `<location>` values exist |

---

## Dashboard Access (If Applicable)

| Setting        | Value                                      |
| -------------- | ------------------------------------------ |
| Host URL       | `https://<your-ip>:5601`                   |
| Default Port   | `5601`                                     |
| Default User   | `admin` (change after setup)               |
| Access Control | Set users/roles via the dashboard settings |

---

## Pro Tips

* Always **back up** config files before editing.
* Use **lab/test environment** before applying in production.
* Keep rules and decoders organized with clear names and comments.
* Restart manager after **any config or rules change**.

---

This file serves as a quick reference for configuring and managing Wazuh. If you're stuck, check logs, validate XML syntax, and go step by step.

