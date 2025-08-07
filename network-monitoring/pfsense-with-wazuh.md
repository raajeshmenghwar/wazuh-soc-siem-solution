# Integrating pfSense Firewall Logs with Wazuh SIEM

Before we begin the installation and configuration process, it is essential to understand some foundational concepts—namely **what a firewall is** and **what pfSense does**. This will give you a clearer perspective on the importance of integrating pfSense with Wazuh for log collection and security monitoring.

## What is a Firewall?

A **firewall** is a network security system—either software-based, hardware-based, or a combination of both—that **monitors and filters network traffic**. It operates on the basis of **predefined security rules**, and its primary function is to **permit or block data packets** between trusted and untrusted networks.

Think of a firewall as a **gatekeeper** or **security checkpoint** between your internal private network and the outside world (like the internet). It ensures that only authorized traffic is allowed in or out.

> **Need a deeper dive into Firewalls and Next-Gen Firewalls (NGFW)?**
> Check out my detailed article on Medium: [Deep Dive into Firewalls](https://medium.com/@rajeshmenghwar)

## What is pfSense?

**pfSense** is a **free and open-source firewall and router software** based on **FreeBSD**. It is designed to deliver **enterprise-grade features** for securing networks in **home labs**, **small businesses**, and **corporate environments**.

### Key Features of pfSense:

* Stateful packet inspection
* VPN (IPSec, OpenVPN)
* DNS/DHCP services
* Network Address Translation (NAT)
* Logging and alerting
* Traffic shaping
* IDS/IPS (via Suricata or Snort)

> Developed by **Electric Sheep Fencing LLC**, pfSense has grown into one of the most trusted solutions in the open-source community.

> **Want to learn more about pfSense setup and use cases?**
> Visit my GitHub documentation: [Working with Firewalls - pfSense](https://github.com/raajeshmenghwar/working-with-firewalls/tree/main/pfsense)

## Why Integrate pfSense with Wazuh?

The integration of pfSense with **Wazuh**, a powerful open-source **Security Information and Event Management (SIEM)** platform, allows you to:

* Collect real-time firewall logs
* Monitor network traffic anomalies
* Detect unauthorized access attempts
* Centralize and correlate log data

This integration enhances **network visibility** and allows **proactive threat detection**.

## Prerequisites

To follow this setup, you must have the following:

* A virtualized environment (VMware/VMware ESXi)
* A working Wazuh Stack:

  * Wazuh Manager
  * Wazuh Indexer
  * Wazuh Dashboard
* A pfSense virtual machine installed and configured

## Step-by-Step Integration Guide

### Step 1: Boot Your Virtual Machines

1. Power on your **pfSense** VM.
2. Boot your **Linux VM** (e.g., Ubuntu).
3. Start your **Wazuh VM** and log in using the default credentials (e.g., `admin/admin`).

### Step 2: Access the Wazuh Dashboard

* On your Linux VM, open a browser and navigate to the Wazuh Dashboard using its IP.
* If you don't know the IP, run `ip a` on the Wazuh VM to find it.

### Step 3: Access the pfSense Web Interface

* Identify the LAN IP of your pfSense VM.
![pfsense-LAN-IP](../assets/pfsense-lan-ip.png)
* Open it in your browser. You may see a certificate warning—click **Advanced** > **Accept the Risk and Continue**.
![pfsense-brower-UI](../assets/pfsense-brower-UI.png)
* Login with:

  * **Username**: `admin`
  * **Password**: `pfsense`

### Step 4: Change pfSense Default Admin Password

* Click the **"Change the password"** warning prompt.
* Set a secure password for the admin account and click **Save**.

### Step 5: Set the Timezone in pfSense

* Navigate to **System > General Setup**.
* Scroll down and set your local **Time Zone**.
* Click **Save**.

Setting the correct time zone helps maintain consistency in log timestamps for analysis.

### Step 6: Enable Syslog Forwarding on pfSense

1. Navigate to **Status > System Logs**.
![pfsense-status](../assets/pfsense-status.png)
2. Click on the **Settings** tab.
![pfsense-status-settings](../assets/pfsense-status-settings.png)
3. Change the **Log Message Format** to `syslog`.
4. (Optional) Enable **Reverse Display** to show newest logs first.
![pfsense-status-settings-2](../assets/pfsense-status-settings-2.png)
5. Enable **Send log messages to remote syslog server**.
6. Set **Source Address** to `LAN`.
![pfsense-status-settings-remote-syslog-server](../assets/pfsense-status-settings-remote-syslog-server.png)
### Step 7: Configure Remote Syslog Server

* In the **Remote log servers** field, enter your Wazuh Manager IP and port like this:

```
x.x.x.x:514
```

> The port **514/UDP** is used for sending syslog messages.

* Under **Remote Syslog Contents**, enable:

  * System Events
  * Firewall Events
  * DNS Events
  * DHCP Events

> ⚠️ Avoid selecting "Everything" as it can overwhelm Wazuh with excessive logs.

* Click **Save**.

![pfsense-status-settings-remote-server](../assets/pfsense-status-settings-remote-server.png)

### Step 8: Create a Firewall Rule for Syslog Traffic

1. Go to **Firewall > Rules > LAN**.
![firewall-rules](../assets/firewall-rules.png)
2. Click the green **Add** button to create a new rule at the top.
![firewall-rules-LAN](../assets/firewall-rules-LAN.png)
3. Configure the rule:

   * **Protocol**: `UDP`
   * **Source**: `LAN Subnet`
   * **Destination**: `Single host` → your **Wazuh IP**
   * **Destination Port**: `514 (syslog)`
   * **Description**: `Forward syslog entries to Wazuh`
4. Click **Save**, then **Apply Changes**.
![firewall-rules-LAN-conf](../assets/firewall-rules-LAN-conf.png)

### Step 9: Configure Wazuh to Listen for Syslog Messages

1. In your browser, login to the **Wazuh Dashboard**.
2. Go to **Server Management > Settings**.
3. Click **Edit Configuration**.
4. Add the following XML block to enable Wazuh to listen for syslog input:

```xml
<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>udp</protocol>
  <allowed-ips>x.x.x.x/24</allowed-ips>
  <local_ip>x.x.x.x</local_ip>
</remote>
```

> * Replace `x.x.x.x/24` with your pfsense subnet (e.g., `192.168.1.0/24`)
> * Replace `x.x.x.x` under `local_ip` with the IP of your Wazuh Manager

5. Click **Save** and then **Restart Manager** > **Confirm**.

### Step 10: Test the Integration

* Log out from the pfSense Web GUI.
* Try logging in again with **incorrect credentials** to generate a failed login event.
* Head back to the **Wazuh Dashboard**:

  * Go to **Explore > Discover**
  * Look for the failed login entry under the logs
![wazuh-pfsense-alerts](../assets/wazuh-pfsense-alerts.png)

If you see the failed login event captured by Wazuh, your integration is successful!

## Summary

By integrating **pfSense** with **Wazuh**, you've enabled powerful network-level visibility within your SOC/SIEM setup. The logs collected from pfSense will now be centrally analyzed by Wazuh, allowing for:

* Better incident response
* Correlated detection of anomalies
* Centralized firewall log management

## 🔗 References and Further Reading

* [Wazuh Documentation](https://documentation.wazuh.com)
* [pfSense Official Docs](https://docs.netgate.com/pfsense/en/latest/)
* [Deep Dive into Firewalls (Rajesh Menghwar - Medium)](https://medium.com/@rajeshmenghwar)
* [GitHub: Working with Firewalls - pfSense](https://github.com/raajeshmenghwar/working-with-firewalls/tree/main/pfsense)

