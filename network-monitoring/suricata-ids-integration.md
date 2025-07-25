iF you don't have rules and IDS or IPS is nothing more than a device.
Step 1: Installing Suricata and Rules
Install Suricata on the Ubuntu endpoint. We tested this process with version 6.0.8 and it can take some time:
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
Download and extract the Emerging Threats Suricata ruleset:
cd /tmp/ && curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz && sudo mkdir /etc/suricata/rules && sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules
Restart the Suricata service:
sudo systemctl restart suricata

Verify Suricata installation
suricata -V
This is Suricata version 8.0.0 RELEASE

nano /etc/suricata/suricata.yaml 


Add the following configuration to the /var/ossec/etc/ossec.conf file of the Wazuh agent. This allows the Wazuh agent to read the Suricata logs file:
<ossec_config>
  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>
</ossec_config>
Step 3: Simulate Attack using Kali Linux
On Kali Linux terminal, run a SYN scan:

nmap -sS -T4 <target-ip>
-sS: SYN scan

-T4: Faster scan timing

This scan should trigger Suricata detection rules for port scanning.

Step 4: View Alerts in Wazuh Dashboard
Login to Wazuh Dashboard.

Navigate to Security Events → Choose agent.

Filter by Rule Group: Suricata

Look for alert like:

ET SCAN Nmap Synchronous Scan