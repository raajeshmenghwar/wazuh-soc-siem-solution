# Maintaining Branding Through Updates and Advanced Tips

This section outlines strategies to ensure your branding is not lost during upgrades and includes optional advanced customization tips.

---

## 1. Preserve Changes Across Wazuh Upgrades

Wazuh updates may overwrite branding files. To prevent data loss:

### Create Backup Directory

```bash
mkdir -p /opt/wazuh-dashboard-backups/assets
```

### Copy Branded Assets

```bash
cp -r /usr/share/wazuh-dashboard/src/core/server/core_app/assets/* /opt/wazuh-dashboard-backups/assets/
cp /usr/share/wazuh-dashboard/plugins/securityDashboards/target/public/*.svg /opt/wazuh-dashboard-backups/
```

---

## 2. Use Git to Track Changes

Optionally, you can initialize a Git repository in the dashboard directory:

```bash
cd /usr/share/wazuh-dashboard
git init
git add .
git commit -m "Initial Wazuh dashboard state"
```

After branding:

```bash
git add .
git commit -m "Applied custom branding"
```

This allows easy comparison and recovery after upgrades.

---

## 3. Optional: Customize Theme Colors (Advanced)

For deeper styling changes (e.g., color palettes), you can modify SCSS/CSS files:

```bash
cd /usr/share/wazuh-dashboard/src/core/server/core_app/styles
```

You can override variables such as:

```scss
$euiColorPrimary: #1e90ff;
$euiColorAccent: #ff9900;
```

Note: This may require rebuilding the frontend using Node.js and yarn.

---

## 4. Reference Links

- [Wazuh UI Customization Guide](https://documentation.wazuh.com/current/user-manual/capabilities/dashboard.html)
- [OpenSearch Dashboards Branding](https://opensearch.org/docs/latest/dashboards/configuration/branding/)
- [Real Favicon Generator](https://realfavicongenerator.net/)
- [Free Image Hosting (ibb.co)](https://ibb.co/)
