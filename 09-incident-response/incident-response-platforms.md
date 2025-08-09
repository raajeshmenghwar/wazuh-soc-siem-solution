Understanding the Role of TheHive & Cortex in SOC
- Wazuh → Detects & collects logs, creates alerts.
- TheHive → Incident Response Platform (IRP) — where you manage and investigate alerts.
- Cortex → Analysis & enrichment engine — it takes IOCs (IPs, hashes, URLs) and enriches them with threat intelligence sources (VirusTotal, AbuseIPDB, MISP, etc.).

In home lab:
- Wazuh = the “security camera”
- TheHive = the “security control room” where the guard logs incidents & investigates
- Cortex = the “forensics lab” that checks suspicious evidence

Where Companies Deploy This
- In real-world SOCs:
- Wazuh Manager → one (or more) dedicated servers.
- TheHive + Cortex → separate IR platform servers.
- They connect via APIs and secured networks.

Analysts get alerts from Wazuh → TheHive automatically creates cases → Cortex enriches → analysts investigate.

So in cyber:

- Wazuh detects a suspicious login from Russia at 3 AM.
- It sends this alert automatically to TheHive.
- TheHive creates a “Case” with all alert details.
- Cortex runs enrichment analyzers (VirusTotal, AbuseIPDB).
- Analyst sees results, confirms if it’s real, documents actions (block IP, reset password).
- Case is closed and archived for compliance.