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

1. Change the HOME_NET with your own iP Address, 
  simple open a new terminal/tab and fire the command ifconfig also note the interface aslo
  in my case my interface is ens33, and ip address is <YOUR_IP_ADDRESS>
2. Just say any instead of home_net in EXTERNAL_NET, so what are we doing, just commeting the #EXTERNAL_NET: "!$HOME_NET"
    and uncommenting the EXTERNAL_NET: "any"
    it'll look like this 
    #EXTERNAL_NET: "!$HOME_NET"
    EXTERNAL_NET: "any"
3. Set the interface for the suricata, remember we noted our interface earlier i.e ens33(in my case), OR you check it again using ifconfig.
  search for the af-packet in /etc/suricata/suricata.yaml, in nano, press the ctrl+w and search for af-packcet and change the ineterface for us.
  By default interface is interface: eth0
  and we have to change it to our own i.e ens33

4. Make to sure the defualt rule path for the suricata, 
  default rule in suricata is : default-rule-path: /var/lib/suricata/rules
  How to search for that in suricata.yaml, in nano simple press the ctl+w and search for rule-path and hit enter:
  rule-files:
    - suricata.rules
    


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