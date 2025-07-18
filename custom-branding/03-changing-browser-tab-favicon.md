# Changing the Browser Tab Favicon

This guide covers replacing the favicon used by the Wazuh Dashboard in browser tabs.

---

## 1. Navigate to the Favicon Directory

The favicon files are stored in:

```bash
cd /usr/share/wazuh-dashboard/src/core/server/core_app/assets/favicons
```

---

## 2. Backup Existing Favicon Files

Before replacing the default icons, create backups:

```bash
for file in *; do mv "$file" "$file.bak"; done
```

This renames each file by appending `.bak` to preserve the originals.

---

## 3. Generate Custom Favicon Package

Use the following tool to generate a complete favicon set:
- [https://realfavicongenerator.net/](https://realfavicongenerator.net/)

Steps:
1. Upload your logo.
2. Download the ZIP package containing generated icons.
3. Extract it:

```bash
unzip favicon_package.zip
```

4. Copy the new files into the `favicons/` directory:

```bash
cp favicon.ico *.png *.xml *.json /usr/share/wazuh-dashboard/src/core/server/core_app/assets/favicons/
```
![Favicon](../assets/Change_Favicon.png)
---

## 4. Restart the Wazuh Dashboard

```bash
sudo systemctl restart wazuh-dashboard
```

The new favicon should now appear in your browser tab.

