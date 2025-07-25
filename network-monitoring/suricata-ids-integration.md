# Suricata Integration with Wazuh SIEM

## Overview

**[Suricata](https://suricata.io/)** is a high-performance, open-source network threat detection engine developed by the [Open Information Security Foundation (OISF)](https://oisf.net). It functions as:

* **IDS (Intrusion Detection System)**
* **IPS (Intrusion Prevention System)**
* **NSM (Network Security Monitoring) tool**

It analyzes network traffic using deep packet inspection (DPI), pattern matching, and protocol analysis, powered by customizable rule sets.


## Suricata vs Other Network Monitoring Tools

[Suricata](https://suricata.io/) is a modern, high-performance engine that acts as an IDS, IPS, and Network Security Monitoring (NSM) tool. It supports multithreading, deep packet inspection, and is designed for scalability and speed.

[Snort](https://snort.org/) is a widely used IDS/IPS focused on real-time traffic analysis and packet logging. While robust and mature, it is single-threaded, which can limit performance on multi-core systems.

[Zeek](https://zeek.org/) (formerly Bro) is primarily a network security monitoring platform. It excels at protocol analysis and behavioral monitoring, providing rich context and scripting capabilities for advanced network visibility.
---

## Why Use Suricata with Wazuh?

| Feature                  | Suricata Role                            | Wazuh Role                                   |
| ------------------------ | ---------------------------------------- | -------------------------------------------- |
| Network Threat Detection | Analyzes traffic and detects anomalies   | Collects and normalizes Suricata logs        |
| Alert Correlation        | Matches traffic with threat intelligence | Aggregates alerts, enriches context          |
| Visibility               | Monitors gateway and server traffic      | Displays findings in a centralized dashboard |

Wazuh integrates Suricata logs into its platform for better correlation, centralized alerting, and event analysis.



---

🛡️ Where Should Suricata Be Installed?
Suricata is a powerful network threat detection engine, but it must be placed strategically in your network to be effective and useful in your Wazuh SIEM setup.

✅ Recommended Installation Locations:
Wazuh Manager
Place Suricata on the same host as the Wazuh Manager. This allows Wazuh to easily access and process Suricata's alerts (from /var/log/suricata/fast.log or /eve.json) without needing external log forwarding.

DMZ Nodes / Perimeter Servers / Bastion Hosts / Web Servers
These are high-exposure points where incoming and outgoing traffic should be monitored closely. Installing Suricata here helps detect scanning, malware, or C2 activities before they reach internal assets.

❌ Not Recommended (unless necessary):
Normal Endpoints / Workstations / Desktops
Do not install Suricata on every endpoint. It can cause confusion and unnecessary resource consumption.

Only consider it when:

You're running high-risk services on the host

You need deep packet inspection at the endpoint level

You have the infrastructure to manage, ship, and analyze logs from multiple agents manually

⚠️ Why This Matters:
 Important Note:
Many beginners make the mistake of installing Suricata on a Wazuh agent (endpoint) and expect logs to show up in the Wazuh dashboard automatically.
That won't happen unless the logs are explicitly shipped back to the Wazuh Manager, which requires additional configuration (like Filebeat or Logstash).

To avoid unnecessary complexity:
➡️ Install Suricata where it makes sense — near network entry/exit points or your Wazuh Manager — not everywhere.

---

## Step 1: Install Suricata on Ubuntu

### 1.1 Add the Official PPA and Install

```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
```

Verify the installation:

```bash
suricata -V
# Output: Suricata version 8.0.0 RELEASE (or similar)
```

---

## Step 2: Download and Configure Rules

### 2.1 What Are Emerging Threats Rules?

[Emerging Threats (ET)](https://rules.emergingthreats.net/) provides free, community-maintained IDS/IPS rules compatible with Suricata.

### 2.2 Download and Install ET Rules

```bash
cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
tar -xvzf emerging.rules.tar.gz
sudo mkdir -p /etc/suricata/rules
sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules
```

---

## Step 3: Configure `suricata.yaml`

Edit the main configuration:

```bash
sudo nano /etc/suricata/suricata.yaml
```

### 3.1 Define Network Variables

Find and modify the following:

```yaml
HOME_NET: "<YOUR_IP_ADDRESS>"   # Replace with your server’s IP
# EXTERNAL_NET: "!$HOME_NET"    # Comment it
EXTERNAL_NET: "any"             # Un-comment it
```

To find your IP:

```bash
ip a
```

### 3.2 Configure Network Interface

Search for `af-packet` section and update the `interface`:

```yaml
af-packet:
  - interface: ens33  # Replace with your actual network interface
```

To find your interface:

```bash
ip link show
```

### 3.3 Set Rule Paths

Set the correct rules path and pattern:

```yaml
default-rule-path: /etc/suricata/rules

rule-files:
  - "*.rules"
```

This ensures all `*.rules` files in the directory are loaded.

Save and close the file.

### 3.4 Restart Suricata

```bash
sudo systemctl restart suricata
```

Check status:

```bash
sudo systemctl status suricata
```

---

## Step 4: Integrate Suricata with Wazuh

### 4.1 Log Output from Suricata

Suricata logs traffic events to:

```bash
/var/log/suricata/eve.json
```

### 4.2 Configure Wazuh to Monitor Suricata Logs

Edit the Wazuh configuration:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add the following inside `<ossec_config>`:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

### 4.3 Restart Wazuh Manager

```bash
sudo systemctl restart wazuh-manager
```

Verify services:

```bash
sudo systemctl status wazuh-manager wazuh-dashboard wazuh-indexer
```

---

## Step 5: Simulate a Network Attack

On Kali Linux or another attacking machine:

```bash
nmap -sS -T4 <TARGET-IP>
```

- `-sS` -> SYN Scan (stealth)
- `-T4` -> Faster timing      

This scan should trigger detection alerts in Suricata.

---

## Step 6: View Alerts in Wazuh Dashboard

1. Log in to the Wazuh Dashboard.
2. Navigate to **Security Events**.
3. Select the appropriate agent (where Suricata is installed).
4. Filter by **Rule Group: Suricata**.

Example alert:

```
ET SCAN Nmap Synchronous Scan
```

This confirms that Wazuh is properly ingesting Suricata alerts.

---

## Monitoring Logs via CLI

View Suricata logs:

```bash
sudo cat /var/log/suricata/fast.log
```

View JSON alerts:

```bash
jq '.' /var/log/suricata/eve.json
```

---

## Troubleshooting

| Symptom                     | Possible Cause                 | Solution                                      |
| --------------------------- | ------------------------------ | --------------------------------------------- |
| No logs in `eve.json`       | Suricata not detecting traffic | Check interface, network rules, permissions   |
| `Permission denied` in logs | Suricata can't read rules      | Fix file permissions on `/etc/suricata/rules` |
| `No rules were loaded`      | Incorrect path or rule-files   | Check `default-rule-path` and `*.rules`       |
| Service won’t start         | YAML syntax or invalid config  | Validate `suricata.yaml` using `suricata -T`  |

---

## Summary

* Suricata is a powerful IDS/IPS that integrates seamlessly with Wazuh.
* Rules are critical—without them, Suricata provides no detection.
* The Emerging Threats rule set offers a great baseline.
* Logs are ingested via the `eve.json` file using Wazuh’s `<localfile>` configuration.
* Use the Wazuh Dashboard for easy visibility and alert triage.



Your documentation is very strong — it’s comprehensive, well-structured, and beginner-friendly. That said, here’s a **critical, professional review and edit** to refine clarity, tone, formatting, and technical correctness, making it suitable for internal or published use in a **corporate-grade SIEM documentation portal**.

---

### ✅ **Overall Strengths**

* ✅ Structured into logical steps
* ✅ Beginner-friendly and technically accurate
* ✅ Uses real commands and context
* ✅ Correctly emphasizes critical pain points (e.g., `eve.json`, rule paths)
* ✅ Markdown formatting is clean and organized

---

### 🔧 **Professional Edits and Suggestions**

Below is the **revised and enhanced version** with improvements in structure, clarity, and technical writing tone:

---

# Suricata Integration with Wazuh SIEM

## Overview

[**Suricata**](https://suricata.io/) is a high-performance, open-source network threat detection engine developed by the [Open Information Security Foundation (OISF)](https://oisf.net). It functions as:

* **Intrusion Detection System (IDS)**
* **Intrusion Prevention System (IPS)**
* **Network Security Monitoring (NSM)** tool

Suricata analyzes traffic using deep packet inspection (DPI), pattern matching, and protocol analysis. Its behavior is driven by customizable and community-maintained **rule sets**.

---

## Why Use Suricata with Wazuh?

Wazuh integrates with Suricata to centralize, enrich, and correlate Suricata’s security event logs. This integration allows security teams to detect, investigate, and respond to network-based threats within the Wazuh SIEM platform.

**Use cases include:**

* Detecting reconnaissance activities like port scans
* Triggering alerts based on suspicious packet signatures
* Enriching host-based logs with network-layer visibility

---

## Suricata vs. Other Tools

**How does Suricata compare to other popular NIDS/NSM tools?**

* **Snort**: Mature and widely adopted, but single-threaded (less performant on multi-core systems)
* **Zeek (Bro)**: Focuses on protocol-level behavior analysis and scripting, great for detailed logging and investigation
* **Suricata**: Multithreaded, high-speed engine supporting IDS/IPS/NSM modes with deep packet inspection and community rulesets

---

## Where Should Suricata Be Installed?

Install **Suricata on the same host as the Wazuh Manager**, or on strategic network nodes like DMZ servers, bastion hosts, or perimeter firewalls.

> ⚠️ **Important:**
> Many users get stuck by installing Suricata on a Wazuh **agent** (or endpoint), expecting logs to appear in the dashboard.
> This won't work unless logs are shipped to the Wazuh Manager. Wazuh **expects Suricata logs to be locally available** on the Manager node by default.

**Recommended placement:**

* ✅ Wazuh Manager (for centralized ingestion)
* ✅ Perimeter/DMZ servers
* ❌ Not recommended on every endpoint (unless required)

---

## Step 1: Install Suricata

```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
```

Verify the installation:

```bash
suricata -V
# Output: Suricata version 8.x.x RELEASE
```

---

## Step 2: Install and Configure Rules

### 2.1 Download Emerging Threats (ET) Rules

[Emerging Threats](https://rules.emergingthreats.net/) provides community-maintained IDS/IPS rules compatible with Suricata.

```bash
cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
tar -xvzf emerging.rules.tar.gz
sudo mkdir -p /etc/suricata/rules
sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules
```

---

## Step 3: Configure `suricata.yaml`

Open the configuration:

```bash
sudo nano /etc/suricata/suricata.yaml
```

### 3.1 Define Network Ranges

```yaml
HOME_NET: "<YOUR_IP_ADDRESS>"   # Replace with your actual IP
# EXTERNAL_NET: "!$HOME_NET"    # Comment this line
EXTERNAL_NET: "any"             # Un-comment this line
```

To find your IP:

```bash
ip a
```

### 3.2 Set the Network Interface

Search for the `af-packet` section and set your interface (e.g., `ens33`):

```yaml
af-packet:
  - interface: ens33
```

To list interfaces:

```bash
ip link show
```

### 3.3 Set Rule Paths

Ensure Suricata loads your rules:

```yaml
default-rule-path: /etc/suricata/rules

rule-files:
  - "*.rules"
```

> This tells Suricata to load **all `.rules` files** in `/etc/suricata/rules`.

### 3.4 Restart Suricata

```bash
sudo systemctl restart suricata
sudo systemctl status suricata
```

---

## Step 4: Integrate Suricata with Wazuh

### 4.1 Suricata Log Path

Suricata logs in JSON format to:

```
/var/log/suricata/eve.json
```

### 4.2 Configure Wazuh to Ingest Logs

Edit the Wazuh Manager config:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add inside `<ossec_config>`:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

### 4.3 Restart Wazuh Manager

```bash
sudo systemctl restart wazuh-manager
```

Verify services:

```bash
sudo systemctl status wazuh-manager wazuh-dashboard wazuh-indexer
```

---

## Step 5: Simulate Detection (Optional Testing)

From a Kali Linux attacker machine:

```bash
nmap -sS -T4 <TARGET-IP>
```

Explanation:

* `-sS`: SYN scan (stealth)
* `-T4`: Fast scan timing

This should trigger Suricata alerts for **Nmap scanning**.

---

## Step 6: View Alerts in the Wazuh Dashboard

1. Open **Wazuh Dashboard**
2. Navigate to **Security Events**
3. Select the agent running Suricata
4. Filter by **Rule Group: Suricata**

Example alert:

```
ET SCAN Nmap Synchronous Scan
```

---

## Monitor Suricata Logs (CLI)

View plain-text alerts:

```bash
sudo cat /var/log/suricata/fast.log
```

View structured JSON logs:

```bash
jq '.' /var/log/suricata/eve.json
```

Install `jq` if needed:

```bash
sudo apt install jq
```

---

## Troubleshooting

| Issue                      | Cause                             | Solution                                    |
| -------------------------- | --------------------------------- | ------------------------------------------- |
| No alerts/logs in Wazuh    | Suricata not logging or misconfig | Check interface, rule paths, log format     |
| `eve.json` empty           | No matching rules triggered       | Simulate an attack (e.g., `nmap`)           |
| Permission denied on rules | Incorrect file permissions        | Use `chmod 640` on rule files               |
| Service failed to start    | YAML syntax error                 | Test config with `suricata -T -c /path/...` |

---

## Summary

* **Suricata** enhances Wazuh by providing deep network visibility via IDS/IPS capabilities.
* Install Suricata **on the same host as Wazuh Manager** for seamless integration.
* Use **Emerging Threats rulesets** for baseline coverage.
* Configure `suricata.yaml` carefully: network interface, IP ranges, and rules path.
* Ingest logs from `eve.json` using Wazuh’s `<localfile>` directive.
* Validate integration via Nmap and the Wazuh dashboard.

---

## Next Steps

After validating Suricata integration:

* Tune detection rules for your environment
* Integrate additional threat intelligence sources
* Consider extending to perimeter devices (e.g., pfSense, network firewalls)
* Monitor performance impact on high-throughput nodes

