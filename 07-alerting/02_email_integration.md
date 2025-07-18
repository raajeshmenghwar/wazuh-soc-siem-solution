## Email Alerting via Postfix in Wazuh

Setting up email alerts in Wazuh helps you stay informed about critical events in your environment. This guide outlines how to configure Wazuh to send professional-grade email alerts.

---

## Requirements

* Wazuh Manager
* A working internet connection
* A Gmail (or similar SMTP-supported) account with App Password configured
* A basic Postfix setup

---

## What is Postfix?

Postfix is a Mail Transfer Agent (MTA) used to route and deliver email. In our case, Postfix will act as the local relay to securely send alert emails using a third-party SMTP service (like Gmail).

### Why Postfix?

* Lightweight and secure
* Open-source and widely supported
* Easy integration with Wazuh

### Alternatives to Postfix

* **Sendmail**: A more complex and older MTA
* **Exim**: Popular on Debian systems
* **MSMTP**: Lightweight SMTP client

> Postfix is recommended due to its stability and simplicity.

---

## Step-by-Step Setup

### 1. Install Postfix

```bash
sudo apt update
sudo apt install postfix mailutils -y
```

> During installation, choose "Internet Site" and use your system's FQDN as the mail name.

---

### 2. Configure Gmail App Password

Since Gmail restricts direct SMTP access from third-party apps:

1. Go to [Google Account > Security](https://myaccount.google.com/security)
2. Enable 2FA (Two-Factor Authentication)
3. Generate an **App Password** under "App Passwords"
4. Save this password securely (do **not** share it)

> 📸 *Screenshot: App password creation step*

---

### 3. Configure Postfix to Use SMTP

Edit or create the file `/etc/postfix/sasl_passwd`:

```ini
[smtp.gmail.com]:587 alert@indussystems.com:your_app_password_here
```

> 📸 *Screenshot: Sample configuration*

Then run:

```bash
sudo postmap /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd*
sudo systemctl restart postfix
```

---

### 4. Add Relay Settings to `main.cf`

Edit `/etc/postfix/main.cf` and add:

```conf
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
```

Restart Postfix:

```bash
sudo systemctl restart postfix
```

> 📸 *Screenshot: Final `main.cf` entries*

---

### 5. Test Mail

```bash
echo "This is a test mail" | mail -s "Wazuh Email Test" reviewer_email@gmail.com
```

If received successfully, your Postfix is working.

---

## Configure Wazuh to Send Email Alerts

### 1. Modify `ossec.conf`

Add inside the `<global>` section:

```xml
<email_notification>yes</email_notification>
<email_to>reviewer_email@gmail.com</email_to>
<email_from>alert@indussystems.com</email_from>
<smtp_server>127.0.0.1</smtp_server>
```

> 📸 *Screenshot: Modified `ossec.conf` email section*

Restart the manager:

```bash
sudo systemctl restart wazuh-manager
```

---

### 2. Trigger an Alert (EICAR or Failed SSH Login)

To test:

```bash
echo "X5O!P%@AP[4\PZX54(P^)7CC)7}\$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!" > /tmp/eicar.com
```

Or try failed SSH login attempts:

```bash
ssh fakeuser@localhost
```

---

## Notes on Email Format

The default Wazuh alert emails are plaintext and not well-formatted. For a more professional look:

* Use a Python script in `/var/ossec/integrations` to format email (See `05_custom_python_alerting.md`)
* Customize headers and footers
* Add logos, styled HTML content

> 📸 *Screenshot: Sample custom email alert*

---

Continue with [Slack Integration](03_slack_integration.md) or [Telegram Bot Integration](04_telegram_integration.md) for multi-channel alerts.
