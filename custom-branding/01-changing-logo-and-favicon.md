# Custom Branding: Changing Logo and Favicon

This guide walks you through replacing the Wazuh Dashboard's default logo and favicon with your own branding assets.

---

## 1. Replace Dashboard Logo

### Location
```bash
cd /usr/share/wazuh-dashboard/plugins/securityDashboards/target/public/
````

Inside this directory, you will find a `.svg` file (e.g., `30e500f584235c2912f16c790345f966.svg`) that serves as the default logo.

### Backup Original

```bash
mv 30e500f584235c2912f16c790345f966.svg 30e500f584235c2912f16c790345f966.svg.bak
```

### Replace with Custom Logo

Download or copy your own `.svg` logo and rename it to match the original filename:

```bash
mv my-custom-logo.svg 30e500f584235c2912f16c790345f966.svg
```
![Replace Dashboard Logo](../assets/Replace_Dashboard_Logo.png)

---
# Custom Branding: Changing Logo and Favicon

This guide walks you through replacing the Wazuh Dashboard's default logo and favicon with your own branding assets.

---

## 1. Replace Dashboard Logo

### Location

```bash
cd /usr/share/wazuh-dashboard/plugins/securityDashboards/target/public/
```

Inside this directory, you will find a `.svg` file (e.g., `30e500f584235c2912f16c790345f966.svg`) that serves as the default logo.

### Backup Original

```bash
mv 30e500f584235c2912f16c790345f966.svg 30e500f584235c2912f16c790345f966.svg.bak
```

### Replace with Custom Logo

Download or copy your own `.svg` logo and rename it to match the original filename:

```bash
mv my-custom-logo.svg 30e500f584235c2912f16c790345f966.svg
```

---

## 2. Change Favicon (Browser Tab Icon)

### Location

```bash
cd /usr/share/wazuh-dashboard/src/core/server/core_app/assets/favicons
```

### Backup Existing Favicons

Instead of moving them to another directory, simply rename each file with a `.bak` extension:

```bash
for file in *; do mv "$file" "$file.bak"; done
```

### Generate and Use Custom Favicons

Use [https://realfavicongenerator.net/](https://realfavicongenerator.net/) to upload your logo and generate a favicon package.

1. Upload your logo on the site.
2. Download the `.zip` file containing the generated favicons.
3. Unzip the package:

   ```bash
   unzip favicon_package.zip
   ```

4. Copy the relevant favicon files into the `favicons/` directory:

   ```bash
   cp favicon.ico android-chrome*.png apple-touch-icon*.png mstile-*.png /usr/share/wazuh-dashboard/src/core/server/core_app/assets/favicons/
   ```

> Ensure the filenames of the new icons match the original ones, or the dashboard may not load them correctly.

---

## 3. Restart Wazuh Dashboard

After making the changes:

```bash
sudo systemctl restart wazuh-dashboard
```
