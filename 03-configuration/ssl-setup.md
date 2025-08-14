# SSL Setup for Secure Manager-Agent Communication

This guide explains how to configure secure SSL/TLS communication between the Wazuh Manager and its agents using either self-signed or CA-issued certificates.

## Why SSL Matters

Using SSL/TLS for manager-agent communication ensures:

- **Confidentiality:** Encrypts all data exchanged between agents and the manager, protecting it from interception.
- **Authenticity:** Ensures that both the agent and manager can verify each other's identities, preventing impersonation.
- **Integrity:** Detects tampering or corruption of transferred data.

## Self-Signed vs CA-Issued Certificates

| Certificate Type     | Use Case                     | Pros                              | Cons                                 |
|----------------------|------------------------------|-----------------------------------|--------------------------------------|
| **Self-Signed**      | Internal labs, testing       | Free, quick to generate           | Must be manually trusted on agents   |
| **CA-Issued**        | Production, enterprise       | Widely trusted, scalable          | May cost money, requires CSR process |

> *For lab setups, self-signed certificates are usually sufficient.*


## Certificate Storage Path

All generated keys and certificates should be placed under:

```
/var/ossec/etc/sslmanager/
```

Recommended structure:

```
/var/ossec/etc/sslmanager/
├── manager-cert.pem
├── manager-key.pem
├── agent-cert.pem       (optional)
└── ca.pem               (optional for CA trust)
````

## Generating a Self-Signed Certificate (Manager Side)

1. Create the SSL directory (if not already present):

```bash
sudo mkdir -p /var/ossec/etc/sslmanager
````

2. Generate a self-signed certificate:

```bash
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout /var/ossec/etc/sslmanager/manager-key.pem \
  -out /var/ossec/etc/sslmanager/manager-cert.pem \
  -subj "/C=PK/ST=Sindh/L=Khairpur/O=SOC Lab/OU=CyberSec/CN=wazuh-manager.local"
```

> Keep your private key (`manager-key.pem`) **secure and readable only by root**

## Configuring `ossec.conf` for SSL

On the **Manager**:

Edit `/var/ossec/etc/ossec.conf` and configure the `<remote>` block:

```xml
<remote>
  <connection>secure</connection>
  <port>1514</port>
</remote>
```

> `secure` instructs the manager to use TLS for agent communication.

On the **Agent**:

Edit `/var/ossec/etc/ossec.conf` to trust the manager’s cert:

```xml
<client>
  <server-ip>192.168.1.100</server-ip>
  <port>1514</port>
  <protocol>tcp</protocol>
  <verify-host>yes</verify-host>
  <ca-verification>yes</ca-verification>
  <ca-path>/var/ossec/etc/sslmanager/manager-cert.pem</ca-path>
</client>
```

> ⚠️ Copy `manager-cert.pem` from the manager to each agent’s `/var/ossec/etc/sslmanager/` directory.


## Restart Services

After configuration:

### On Manager:

```bash
sudo systemctl restart wazuh-manager
```

### On Agent:

```bash
sudo systemctl restart wazuh-agent
```

## Verifying the Connection

Check Wazuh logs to confirm secure connection:

```bash
# On Manager
sudo tail -f /var/ossec/logs/ossec.log

# On Agent
sudo tail -f /var/ossec/logs/ossec.log
```

Look for log lines like:

```
INFO: Secure connection established with agent '192.168.1.150'
```

## Troubleshooting SSL Issues

| Issue                            | Cause                                  | Solution                                    |
| -------------------------------- | -------------------------------------- | ------------------------------------------- |
| `SSL handshake failed`           | Invalid or unmatched certs             | Ensure agent has correct `manager-cert.pem` |
| `No such file or directory`      | Wrong path in config                   | Verify cert file paths on both sides        |
| `Connection refused`             | Firewall or wrong port                 | Ensure port 1514 is open and in use         |
| Agent not appearing in dashboard | Cert accepted but communication failed | Restart both agent and manager; review logs |

## Optional: Using CA-Issued Certificates

1. Generate a **Certificate Signing Request (CSR)**.
2. Send CSR to your Certificate Authority (internal or public).
3. Receive and install signed cert + CA bundle.
4. Place certs in `/var/ossec/etc/sslmanager/` and update paths in `ossec.conf` on both sides.

> This is more secure and scalable for production deployments.

## Summary

* SSL encrypts agent-manager communication and is highly recommended.
* Use self-signed certs for labs; use CA-signed for production.
* Don't forget to restart Wazuh services after changes.
* Always verify logs to confirm successful handshake.

For further help, visit the [official Wazuh SSL guide](https://documentation.wazuh.com/current/user-manual/agents/agent-connection.html#secure-connection).

**📖 Read Next:** [04 – Office 365 Logs Integration](../04-log-sources/office365-logs-integration.md) — Collecting and parsing Office 365 logs in Wazuh.
