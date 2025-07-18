# Email Alerting Integration with Wazuh

This guide explains how to configure Wazuh to send alerts via email using Postfix and Gmail SMTP.

---

## Prerequisites

- A working Wazuh Manager
- Postfix installed on the server
- Gmail account (preferably a dedicated alerting account)

---

## ✉️ Step-by-Step Setup

### 1. Configure Gmail SMTP Credentials

Create `/etc/postfix/sasl_passwd`:

```bash
[smtp.gmail.com]:587 raajeshkumar2109@gmail.com:your_app_password
````

Replace `your_app_password` with a valid **App Password** generated from Google.

### 2. Secure the Credentials

```bash
sudo postmap /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd*
```

### 3. Update Postfix Configuration

Append to `/etc/postfix/main.cf`:

```ini
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
smtp_use_tls = yes
```

Restart Postfix:

```bash
sudo systemctl restart postfix
```

---

## Test Email Functionality

```bash
echo "Test email from Wazuh" | mail -s "Wazuh Test Email" your_email@gmail.com
```

---

## 🛠️ Configure Wazuh to Send Email Alerts

Edit `/var/ossec/etc/ossec.conf`:

```xml
<global>
  <email_notification>yes</email_notification>
  <email_to>your_email@gmail.com</email_to>
  <smtp_server>127.0.0.1</smtp_server>
  <email_from>wazuh@indussystems.local</email_from>
</global>
```

---

## 📌 Notes

* **Only critical alerts** are sent by default (level ≥ 7).
* You can tune rules for specific alert levels or groups.

---

## 🧪 Test Rule with EICAR

You can use the **EICAR test file** to trigger an alert:

```bash
curl -o eicar.txt https://secure.eicar.org/eicar.com.txt
```

---

## 🎯 Tips

| Tip                | Details                                              |
| ------------------ | ---------------------------------------------------- |
| Use branded domain | e.g., `alerts@indussystems.com` for professionalism. |
| Enable DKIM/DMARC  | For email deliverability.                            |

---

### ✅ `05_custom_python_alerting.md`

```markdown
# Custom Alerting with Python Integrations in Wazuh

Wazuh supports Python-based alerting integrations through its `/var/ossec/integrations/` directory. This is ideal for sending alerts to custom APIs, dashboards, or styled emails.

---

## 📂 Directory Overview

| Path                             | Purpose                         |
|----------------------------------|---------------------------------|
| `/var/ossec/integrations/`       | Stores Python and bash scripts |
| `.py` files                      | Python-based alert integrations |

---

## 🧱 Basic Structure of a Python Integration Script

```python
#!/usr/bin/env python3

import sys
import json

def send_alert(alert_data):
    # Custom logic to send email or webhook
    print("Alert:", alert_data['rule']['description'])

if __name__ == "__main__":
    alert = json.loads(sys.stdin.read())
    send_alert(alert)
````

Make the script executable:

```bash
chmod +x /var/ossec/integrations/my_alert.py
```

---

## 🛠️ Configure ossec.conf

Add integration:

```xml
<integration>
  <name>my_alert</name>
  <hook_url>https://my.custom.url/</hook_url>
  <alert_format>json</alert_format>
  <level>10</level>
</integration>
```

---

## 📧 Advanced: HTML Email Integration with Logo

Use libraries:

```bash
pip install yagmail jinja2
```

Python Example (simplified):

```python
import yagmail
from jinja2 import Template

template = Template(open('template.html').read())
html = template.render(alert=alert_data)

yag = yagmail.SMTP("your_email@gmail.com", "app_password")
yag.send(to="receiver@example.com", subject="Wazuh Alert", contents=html)
```

---

## 📁 Online Templates

* [Stripo.email](https://stripo.email)
* [Mailchimp Templates](https://mailchimp.com/email-templates/)
* [ReallyGoodEmails](https://reallygoodemails.com)

---

## ✅ Tips

* Validate all custom scripts.
* Always set `<alert_format>json</alert_format>`.
* You can reuse data from `/var/ossec/logs/alerts/alerts.json`.
