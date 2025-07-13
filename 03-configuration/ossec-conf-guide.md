### Full breakdown of every major config block in ossec.conf
#### Understanding the `ossec.conf` Structure

The `ossec.conf` file uses XML format. Think of `<ossec_config>` as a parent container holding all configuration blocks (like folders inside a main folder). Each block is wrapped in tags (e.g., `<global>...</global>`). Tags must always be closed, and while indentation helps readability, the tag structure is what matters.

---

## `<global>`
- **Purpose:** Controls general settings such as email notifications, timeouts, and log limits.
- **Example:**
    ```xml
    <global>
        <email_notification>yes</email_notification>
        <email_to>admin@example.com</email_to>
        <logall>no</logall>
    </global>
    ```
- **Explanation:** Enables email alerts, sets recipient, and configures log verbosity.

---

## `<alerts>`
- **Purpose:** Sets alert levels, log locations, and email frequency.
- **Example:**
    ```xml
    <alerts>
        <log_alert_level>3</log_alert_level>
        <email_alert_level>7</email_alert_level>
    </alerts>
    ```
- **Explanation:** Only alerts above these levels are logged or emailed.

---

## `<remote>`
- **Purpose:** Configures agent ↔ manager communication (IP, ports, modes).
- **Modes:** `secure`, `plain`, `hybrid`
- **Example:**
    ```xml
    <remote>
        <connection>secure</connection>
        <port>1514</port>
    </remote>
    ```
- **Explanation:** Sets secure communication on port 1514.

---

## `<active-response>`
- **Purpose:** Automates responses (e.g., blocking IPs) using scripts.
- **Key Tags:** `<location>`, `<rules_id>`
- **Example:**
    ```xml
    <active-response>
        <command>firewalldrop</command>
        <location>local</location>
        <rules_id>1002,1003</rules_id>
    </active-response>
    ```
- **Explanation:** Runs `firewalldrop` locally for specific rule IDs.

---

## `<syscheck>`
- **Purpose:** File integrity monitoring (what to watch, how often).
- **Example:**
    ```xml
    <syscheck>
        <frequency>3600</frequency>
        <directories check_all="yes">/etc,/usr/bin</directories>
    </syscheck>
    ```
- **Explanation:** Scans listed directories every hour.

---

## `<rootcheck>`
- **Purpose:** Detects rootkits and system misconfigurations.
- **Example:**
    ```xml
    <rootcheck>
        <rootkit_files>yes</rootkit_files>
    </rootcheck>
    ```
- **Explanation:** Enables rootkit file checks.

---

## `<localfile>`
- **Purpose:** Monitors custom log files (e.g., Apache, auth logs).
- **Example:**
    ```xml
    <localfile>
        <log_format>syslog</log_format>
        <location>/var/log/auth.log</location>
    </localfile>
    ```
- **Explanation:** Watches `/var/log/auth.log` for syslog-formatted events.

---

## `<auth>`
- **Purpose:** Controls agent authentication (useful for enrolling new agents).
- **Example:**
    ```xml
    <auth>
        <disabled>no</disabled>
    </auth>
    ```
- **Explanation:** Enables agent authentication.

---

## `<cluster>` (if clustering is enabled)
- **Purpose:** Configures master/worker nodes and sync settings.
- **Example:**
    ```xml
    <cluster>
        <name>wazuh-cluster</name>
        <node_type>master</node_type>
    </cluster>
    ```
- **Explanation:** Sets up a cluster named `wazuh-cluster` as master.

---

### Do’s and Don'ts When Editing

- Validate changes with `xmllint` or similar tools  
- Don’t remove critical tags like `<rules>`  
- Keep comments for custom sections for clarity

---

### Reference Table: Common Paths & Files

| Section      | Description              | Related Path                        |
|--------------|-------------------------|-------------------------------------|
| `<rules>`    | Loads detection rules   | `/var/ossec/ruleset/rules/`         |
| `<decoders>` | Loads decoders          | `/var/ossec/etc/decoders/`          |
| `<localfile>`| Custom log files        | `/var/log/auth.log`, etc.           |

---

### Visual Hierarchy Example

```
<ossec_config>
    <global>...</global>
    <alerts>...</alerts>
    <remote>...</remote>
    <active-response>...</active-response>
    <syscheck>...</syscheck>
    <rootcheck>...</rootcheck>
    <localfile>...</localfile>
    <auth>...</auth>
    <cluster>...</cluster>
</ossec_config>
```

---

**Impact:**  
Changes in `ossec.conf` directly affect alerting, agent behavior, and what appears in the dashboard. Always back up before editing and test after changes.