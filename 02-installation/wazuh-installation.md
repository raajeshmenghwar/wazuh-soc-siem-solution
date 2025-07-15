# **Wazuh SIEM Installation Guide (Wazuh 4.x+ All-in-One)**

## **Overview**

**Wazuh** is a leading open-source **Security Information and Event Management (SIEM)** platform that provides unified security visibility and threat detection across endpoints, cloud workloads, and on-premises infrastructure.

With the release of **version 4.x and above**, Wazuh introduced a **fully integrated technology stack**, offering the following core components:

* **Wazuh Manager**: Collects, analyzes, and correlates logs and events.
* **Wazuh Indexer**: Efficiently stores and indexes data (built on OpenSearch; replaces Elasticsearch).
* **Wazuh Dashboard**: A powerful web-based interface for managing alerts and visualizing security data (replaces Kibana).

---

## **Highly Recommended: Quick All-in-One Installation (Ideal for Labs and Quick Deployments)**

The **official Wazuh quick installation script** is the most recommended method for deploying Wazuh in **test or evaluation environments**. It significantly reduces setup time and installs all essential components:

* Wazuh Manager
* Wazuh Indexer (OpenSearch-based)
* Wazuh Dashboard
* Filebeat (log shipper)

### 🔹 **Installation Steps**

```bash
curl -sO https://packages.wazuh.com/4.8/wazuh-install.sh
chmod +x wazuh-install.sh
sudo ./wazuh-install.sh --wazuh-manager --indexer --dashboard
```

> Takes approximately 10–15 minutes depending on system and network speed.

### 🔹 **Access the Wazuh Dashboard**

* Open your browser: `https://<your_server_ip>:5601`
* Default credentials:

  * **Username**: `admin`
  * **Password**: `admin`

**Change the default password** after first login using the CLI:

```bash
/usr/share/wazuh-dashboard/plugins/wazuh/wazuh-passwords-tool.sh -a
```

📘 [Quick Install Script – Official Docs](https://documentation.wazuh.com/current/installation-guide/wazuh-installation-packages/wazuh-install-script.html)

---

## **Why No More Separate Kibana/Elasticsearch?**

Wazuh 4.x+ ships with its **own indexer and dashboard**, removing the need to manually install or configure Elasticsearch and Kibana. This ensures:

* Simplified deployment
* Reduced compatibility issues
* Unified maintenance and support

| Feature/Component | Pre-4.x (ELK Stack)        | Post-4.x (Integrated Stack)   |
| ----------------- | -------------------------- | ----------------------------- |
| Indexing          | Elasticsearch (manual)     | Wazuh Indexer (bundled)       |
| Dashboard UI      | Kibana (manual)            | Wazuh Dashboard (bundled)     |
| Deployment        | Complex (multiple systems) | Simplified (single script)    |
| Updates           | Separate per component     | Unified update management     |
| Support Scope     | Partial                    | Full (end-to-end Wazuh stack) |

---

## **Alternative Manual Installation (Advanced/Production Customization)**

For users who require fine-grained control or production-grade environments, Wazuh can also be installed manually by adding the Wazuh APT repository.

### **Step 1: Add the Wazuh Repository**

```bash
curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
sudo apt-key add GPG-KEY-WAZUH
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt-get update
```

[Repository Setup – Official Docs](https://documentation.wazuh.com/current/installation-guide/packages-list.html)

---

### **Step 2: Install Core Components**

```bash
sudo apt-get install wazuh-manager wazuh-indexer wazuh-dashboard
```

**Roles of Each Component:**

* `wazuh-manager`: Core event collection, rule engine, and correlation.
* `wazuh-indexer`: Stores and searches event data (OpenSearch backend).
* `wazuh-dashboard`: Web UI for visualization and configuration.

---

###  **Step 3: Start and Enable Services**

```bash
sudo systemctl enable --now wazuh-manager
sudo systemctl enable --now wazuh-indexer
sudo systemctl enable --now wazuh-dashboard
```

---

### **Step 4: Access the Dashboard**

Visit: `https://<your_server_ip>:5601`
Default credentials:

* **admin / admin**

Change password securely using:

```bash
/usr/share/wazuh-dashboard/plugins/wazuh/wazuh-passwords-tool.sh -a
```

---

##  **Troubleshooting Tips**

| Problem                          | Recommended Action                                         |
| -------------------------------- | ---------------------------------------------------------- |
| Services not starting            | Use `journalctl -u <service_name>` to inspect logs         |
| Dashboard not accessible         | Confirm port `5601` is open and no firewall is blocking it |
| Port conflicts                   | Check if ports `1514/1515`, `9200`, or `5601` are in use   |
| Certificate issues (self-signed) | Use provided defaults or upload custom certs in production |

---

## **Alternative Installation Methods (For Testing/Labs)**

Wazuh also offers official alternatives to suit different environments:

### [Docker-based Deployment](https://documentation.wazuh.com/current/installation-guide/docker/index.html)

Deploy Wazuh components quickly using Docker Compose. Suitable for CI/CD, dev, or lab environments.

### [OVA Virtual Appliance](https://documentation.wazuh.com/current/installation-guide/ova/index.html)

Download a pre-configured VirtualBox/VMware appliance for plug-and-play deployment.

---

## **References & Resources**

* [Official Wazuh Documentation](https://documentation.wazuh.com/current/index.html)
* [Quick Install Script](https://documentation.wazuh.com/current/installation-guide/wazuh-installation-packages/wazuh-install-script.html)
* [All-in-One Installation Guide](https://documentation.wazuh.com/current/installation-guide/all-in-one/index.html)
* [Distributed Architecture Guide](https://documentation.wazuh.com/current/installation-guide/distributed-deployment/index.html)

