# Custom Branding: Modify Page Title and Header Branding

This section explains how to configure the Wazuh Dashboard title, header logo, and loading screens via configuration files.

---

## 1. Configuration File Location

### Path:
```bash
sudo nano /etc/wazuh-dashboard/opensearch_dashboards.yml
````

In some installations:

```bash
/usr/share/wazuh-dashboard/config/opensearch_dashboards.yml
```

---

## 2. Modify Branding Settings

Add or update the following keys in the config file:

```yaml
opensearchDashboards.branding.applicationTitle: "Your Organization Name"

opensearch_security.ui.basicauth.login.brandimage: "https://yourdomain.com/images/login-logo.png"

opensearchDashboards.branding:
  logo:
    defaultUrl: "https://yourdomain.com/images/logo-light.svg"
    darkModeUrl: "https://yourdomain.com/images/logo-dark.svg"
  loadingLogo:
    defaultUrl: "https://yourdomain.com/images/loading-light.svg"
    darkModeUrl: "https://yourdomain.com/images/loading-dark.svg"
  mark:
    defaultUrl: "https://yourdomain.com/images/mark-light.svg"
    darkModeUrl: "https://yourdomain.com/images/mark-dark.svg"
```

---

## 3. Restart Wazuh Dashboard

```bash
sudo systemctl restart wazuh-dashboard
```

