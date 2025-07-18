# Customizing the Wazuh Login Page Background

This guide explains how to update the login page background image for the Wazuh Dashboard.

---

## 1. Locate the Login Background File

Navigate to the asset directory containing the login page background:

```bash
cd /usr/share/wazuh-dashboard/src/core/server/core_app/assets/
```

The default file is:

```bash
wazuh_login_bg.svg
```

---

## 2. Backup the Original Background

Create a backup of the original file before replacing it:

```bash
mv wazuh_login_bg.svg wazuh_login_bg.svg.bak
```

---

## 3. Replace with a Custom Background

Upload your custom SVG file and rename it:

```bash
mv custom-login-background.svg wazuh_login_bg.svg
```

Make sure the image is in SVG format for full compatibility.

---

## 4. Restart the Wazuh Dashboard

```bash
sudo systemctl restart wazuh-dashboard
```

Reload the login page in your browser. Clear cache if the old background is still displayed.
