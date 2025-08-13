# Integrating Wazuh with TheHive and Cortex — Beginner-Friendly Guide

## Objective

This guide will walk you through **integrating Wazuh (log detection)** with **TheHive (incident management)** and **Cortex (analysis engine)** to create a small but powerful SOC (Security Operations Center) setup.  
We’ll explain the role of each tool, provide real-world scenarios, and guide you step-by-step through installation and configuration — whether you’re setting up a **home lab** or deploying in a **production SOC**.

## 1. Understanding the Role of Each Component in a SOC Environment

Think of your SOC as a **security team for your digital property**.  

- **Wazuh** → *Security Camera System*  
  Watches your network and systems, collecting logs and alerting you to suspicious activities (intrusion detection).

- **TheHive** → *Control Room*  
  Where security analysts manage, track, and investigate alerts (incident management).

- **Cortex** → *Forensics Lab*  
  Takes raw evidence (IP addresses, files, URLs) and runs analysis to give context and enrichment.

### How They Work Together
1. **Detection**: Wazuh spots suspicious activity (e.g., unusual login attempt).  
2. **Case Creation**: Wazuh sends alert → TheHive automatically creates an incident case.  
3. **Enrichment**: TheHive sends suspicious details to Cortex for deeper analysis.  
4. **Investigation**: Analyst reviews enriched results and determines next steps.  
5. **Resolution**: Threat is contained and documented.

---

## 2. Real-World SOC Example Scenario

**Scenario:**  
- **Event**: A suspicious login is detected on a company server.  
- **Wazuh**: Triggers an alert with the source IP address.  
- **TheHive**: Automatically creates a case titled “Suspicious Login Attempt”.  
- **Cortex**: Runs an IP reputation check and finds that the IP is linked to known brute-force attacks.  
- **Analyst**: Uses this info to block the IP at the firewall and document the incident.

In a **home lab**, this might mean detecting a login attempt to your personal VM.  
In a **real SOC**, it could mean protecting thousands of employee accounts.

---

## 3. Beginner-Friendly Explanations of Each Technology

### Wazuh Manager
A **security monitoring tool** that collects logs from endpoints, detects anomalies, and triggers alerts.

### TheHive
An **incident response platform** where you create, manage, and track cases.

### Cortex
An **analysis engine** with "analyzers" (tools) for IP lookup, file hash analysis, malware scanning, etc.

### Docker
A platform that runs software in **isolated containers** so it’s easy to deploy without worrying about dependencies.

### Docker Compose
A tool to define and run multiple Docker containers with a single YAML file.

### REST API Basics
APIs let programs talk to each other.  
Wazuh → TheHive integration works via API calls — Wazuh sends JSON data to TheHive’s REST API to create cases.

---

## 4. Installation & Deployment Methods

You can deploy in several ways depending on your resources:

### Option 1 — VMware (Local Lab)
- **Recommended Specs**:  
  - CPU: 4 cores  
  - RAM: 8 GB (12 GB preferred)  
  - Disk: 50 GB+  
- Install Ubuntu/Debian VM and run Docker inside it.

### Option 2 — Docker/Docker Compose in Existing VM
- Quickest method for labs.
- Lightweight for testing.

### Option 3 — VPS Server
- Choose a provider that supports Docker.
- Minimum specs: 2 CPU, 4 GB RAM, SSD storage.

### Option 4 — Azure VM (Using Credits)
- Use **Azure free credits** to test SOC setup.
- Recommended: Standard B2s or higher for performance.

---

## 5. Step-by-Step Integration

### Step 1 — Install Wazuh
```bash
docker network create wazuh-net
docker run -d --name wazuh-manager --network wazuh-net -p 55000:55000 -p 1514:1514/udp wazuh/wazuh
````

**Why?**
This is like installing your security cameras — without it, nothing is being monitored.

---

### Step 2 — Install TheHive

```yaml
# docker-compose-thehive.yml
version: '3.7'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.24
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms1g -Xmx1g
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - "9200:9200"

  thehive:
    image: strangebee/thehive:latest
    depends_on:
      - elasticsearch
    ports:
      - "9000:9000"
```

**Why?**
TheHive is your control room — you need it to receive and manage alerts.

---

### Step 3 — Install Cortex

```yaml
# docker-compose-cortex.yml
version: '3.7'
services:
  cortex:
    image: strangebee/cortex:latest
    ports:
      - "9001:9001"
```

**Why?**
Cortex is your forensics lab — it investigates data sent from TheHive.

---

### Step 4 — Configure Wazuh → TheHive

Edit Wazuh’s `ossec.conf`:

```xml
<integration>
  <name>thehive</name>
  <hook_url>http://THEHIVE_SERVER:9000/api/alert</hook_url>
  <api_key>YOUR_THEHIVE_API_KEY</api_key>
</integration>
```

**Scenario**: This is like telling your security cameras where to send footage.

---

### Step 5 — Configure TheHive → Cortex

In TheHive UI:

* Go to **Admin → Cortex Configuration**.
* Add Cortex server URL: `http://cortex:9001`.
* Add API key from Cortex.

---

## 6. YAML Examples

### General Deployment

```yaml
version: '3.7'
services:
  wazuh:
    image: wazuh/wazuh
    ports:
      - "55000:55000"
      - "1514:1514/udp"

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.24
    environment:
      - discovery.type=single-node

  thehive:
    image: strangebee/thehive:latest
    ports:
      - "9000:9000"

  cortex:
    image: strangebee/cortex:latest
    ports:
      - "9001:9001"
```

### Lightweight Deployment (Home Lab)

```yaml
version: "2"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.6.0
    environment:
      - http.host=0.0.0.0
      - transport.host=0.0.0.0
      - xpack.security.enabled=false
      - cluster.name=hive
      - script.inline=true
      - thread_pool.index.queue_size=100000
      - thread_pool.search.queue_size=100000
      - thread_pool.bulk.queue_size=100000
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
  cortex:
    image: thehiveproject/cortex:latest
    ports:
      - "0.0.0.0:9001:9001"
  thehive:
    image: thehiveproject/thehive:latest
    depends_on:
      - elasticsearch
      - cortex
    ports:
      - "0.0.0.0:9000:9000"
```

---

## 7. References & Further Reading

* Wazuh Docs: [https://documentation.wazuh.com](https://documentation.wazuh.com)
* TheHive Docs: [https://docs.strangebee.com/thehive](https://docs.strangebee.com/thehive)
* Cortex Docs: [https://docs.strangebee.com/cortex](https://docs.strangebee.com/cortex)

---

**Final Note:**
In a SOC, speed and clarity are everything. This integration ensures that your **detection (Wazuh)**, **incident tracking (TheHive)**, and **analysis (Cortex)** work in harmony — whether you’re protecting a personal lab or an enterprise network.

**📖 Read Next:** [10 – Hardening & Security](../10-hardening-and-security/) — Best practices for securing your Wazuh environment.
