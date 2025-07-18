# Custom Branding: Customizing Login Page Background

This guide details how to update the login background image of the Wazuh Dashboard.

---

## 1. Background File Location
```bash
cd /usr/share/wazuh-dashboard/src/core/server/core_app/assets
````

### Locate Background File

Look for the file:

```bash
wazuh_login_bg.svg
```

### Backup Original File

```bash
mv wazuh_login_bg.svg wazuh_login_bg.svg.bak
```

### Replace with Custom Image

Download or create your own SVG image:

```bash
mv my-background.svg wazuh_login_bg.svg
```

---

## 2. Restart Dashboard to Apply Changes

```bash
sudo systemctl restart wazuh-dashboard
```

> For consistent results, clear browser cache before reloading the login page.