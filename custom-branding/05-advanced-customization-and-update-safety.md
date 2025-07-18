# Advanced Customizations and Branding Persistence

To keep branding changes intact during upgrades and further customize the experience, follow these best practices.

---

## 1. Preserve Changes Across Updates

Wazuh and OpenSearch updates may overwrite UI files. To avoid this:

- Keep a copy of modified assets in a separate backup directory:
  ```bash
  mkdir -p /opt/wazuh-dashboard-backups/assets
  cp -r /usr/share/wazuh-dashboard/src/core/server/core_app/assets/* /opt/wazuh-dashboard-backups/assets/
````

* After upgrades, reapply changes or automate restoration using scripts.

---


## 4. References

* [Wazuh Official Docs – UI Customization](https://documentation.wazuh.com/current/user-manual/capabilities/dashboard.html)
* [OpenSearch Dashboards Branding Guide](https://opensearch.org/docs/latest/dashboards/configuration/branding/)