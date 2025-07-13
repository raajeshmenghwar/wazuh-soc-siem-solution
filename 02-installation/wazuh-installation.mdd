# Wazuh SIEM Installation Guide (Wazuh 4.x+ All-in-One)

## Overview

Wazuh is an open-source Security Information and Event Management (SIEM) platform. Starting from version 4.x, Wazuh provides an integrated solution that includes:

- **Wazuh Manager**: Collects, analyzes, and correlates security data.
- **Wazuh Indexer**: Stores and indexes alerts and events (replaces Elasticsearch).
- **Wazuh Dashboard**: Web interface for monitoring and managing security events (replaces Kibana).

## Why No More Separate Kibana/Elasticsearch?

In Wazuh 4.x+, the platform ships with its own **Wazuh Indexer** and **Wazuh Dashboard**. This eliminates the need to install and configure Elasticsearch and Kibana separately, simplifying deployment and ensuring compatibility.

## Wazuh Setup: Pre-4.x vs Post-4.x

| Feature/Component         | Pre-4.x (Manual ELK)         | Post-4.x (Integrated)         |
|--------------------------|------------------------------|-------------------------------|
| Data Indexing            | Elasticsearch (manual)        | Wazuh Indexer (bundled)       |
| Web UI                   | Kibana (manual)               | Wazuh Dashboard (bundled)     |
| Installation Complexity  | High (multiple components)    | Low (single package)          |
| Updates                  | Separate for each component   | Unified updates               |
| Official Support         | Limited for ELK integration   | Full support for all parts    |

## Step-by-Step Installation (All-in-One)

### 1. Add the Wazuh Repository

```bash
curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
sudo apt-key add GPG-KEY-WAZUH
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt-get update
```

### 2. Install the Wazuh All-in-One Package

```bash
sudo apt-get install wazuh-manager wazuh-indexer wazuh-dashboard
```

- `wazuh-manager`: Core analysis engine
- `wazuh-indexer`: Data storage and search
- `wazuh-dashboard`: Web interface

### 3. Start and Enable Services

```bash
sudo systemctl enable --now wazuh-manager
sudo systemctl enable --now wazuh-indexer
sudo systemctl enable --now wazuh-dashboard
```

### 4. Access the Wazuh Dashboard

- Open your browser and go to: [https://<your-server-ip>:5601](https://<your-server-ip>:5601)
- Default port: **5601**
- Default credentials: `admin` / `admin` (change after first login)

### 5. Troubleshooting Common Startup Errors

- **Service not starting**: Check logs with `sudo journalctl -u wazuh-manager` (or `wazuh-indexer`, `wazuh-dashboard`).
- **Port conflicts**: Ensure ports 5601 (dashboard), 9200 (indexer), and 1514/1515 (manager) are free.
- **Firewall issues**: Open required ports in your firewall.

### 6. Alternative Installations (Lab/Testing)

- **Docker**: Use the official [Wazuh Docker images](https://documentation.wazuh.com/current/installation-guide/installing-wazuh/docker.html) for quick lab setups.
- **OVA Image**: Download a pre-built virtual appliance for rapid deployment in virtualization environments.

## References

- [Wazuh Official Documentation](https://documentation.wazuh.com/current/)
- [Wazuh Installation Guide](https://documentation.wazuh.com/current/installation-guide/index.html)
