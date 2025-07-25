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

## Where Should Suricata Be Installed?

| Install Location               | Recommended | Description                                                 |
| ------------------------------ | ----------- | ----------------------------------------------------------- |
| Wazuh Manager                  | ✅ Yes       | Monitors traffic reaching your SIEM system                  |
| Perimeter Servers / DMZ        | ✅ Yes       | Detects external attacks early                              |
| Internal Workstations / Agents | ❌ Optional  | Not needed unless traffic inspection at host level is vital |

Suricata is not meant to be installed on every endpoint. Deploy it at strategic points where network visibility matters.

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
# EXTERNAL_NET: "!$HOME_NET"
EXTERNAL_NET: "any"
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

| Flag  | Purpose            |
| ----- | ------------------ |
| `-sS` | SYN Scan (stealth) |
| `-T4` | Faster timing      |

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

## Temporarily Stop Suricata

If you need to pause logging:

```bash
sudo systemctl stop suricata
sudo systemctl disable suricata  # Prevent on reboot
```

To resume:

```bash
sudo systemctl enable suricata
sudo systemctl start suricata
```

---

## Summary

* Suricata is a powerful IDS/IPS that integrates seamlessly with Wazuh.
* Rules are critical—without them, Suricata provides no detection.
* The Emerging Threats rule set offers a great baseline.
* Logs are ingested via the `eve.json` file using Wazuh’s `<localfile>` configuration.
* Use the Wazuh Dashboard for easy visibility and alert triage.

---

Let me know if you'd like this exported as a `.md` file or integrated into your documentation repo.
