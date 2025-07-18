# Changing the Wazuh Dashboard Logo

## 1. Navigate to the Logo Directory

The default logo is located inside the Wazuh dashboard plugin directory:

```bash
cd /usr/share/wazuh-dashboard/plugins/securityDashboards/target/public/
```

This folder contains an SVG file (e.g., `30e500f584235c2912f16c790345f966.svg`) used as the Wazuh dashboard logo.

---

## 2. Backup the Original Logo

It is good practice to back up the existing file before making changes:

```bash
mv 30e500f584235c2912f16c790345f966.svg 30e500f584235c2912f16c790345f966.svg.bak
```

This command renames the original file with a `.bak` extension for safekeeping.

---

## 3. Replace with a Custom Logo

Upload your own SVG logo to the server and rename it to the original filename to ensure the dashboard uses it:

```bash
mv my-custom-logo.svg 30e500f584235c2912f16c790345f966.svg
```

Ensure the new logo is in SVG format and matches the dimensions and style of the original.
 
![Replace Wazuh Custom Logo](../assets/Replace_Dashboard_Logo.png) 
---

## 4. Restart the Wazuh Dashboard

To apply the changes:

```bash
sudo systemctl restart wazuh-dashboard
```

This restarts the dashboard service so it can load the new logo.
