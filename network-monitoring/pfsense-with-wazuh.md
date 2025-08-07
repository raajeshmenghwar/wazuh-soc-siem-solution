Before deep diving into the installtion process let's understand first what actually the firewall is? and also what is the pfsense.

Firewall: 
A **firewall** is a security device — either software-based, hardware-based, or both — that monitors and filters incoming and outgoing traffic based on predefined security rules. Its main role is to act as a barrier between trusted internal networks and untrusted external sources, such as the internet.

Think of a firewall as a **security checkpoint** at a building entrance. It only allows entry to people (traffic) who meet specific conditions, blocking the rest.

If you want to read more about firewalls, check out my medium writeup about deep dive into firewalls, NFGW all explained.

https://medium.com/@rajeshmenghwar

Pfsense:


**pfSense** is a free and open-source firewall and router platform built on **FreeBSD**, designed to provide powerful and flexible network security solutions. It is widely used in **home labs**, **small-to-medium businesses**, and even **enterprise environments**, thanks to its rich feature set and reliability.

Originally developed by Electric Sheep Fencing LLC, pfSense has become one of the most trusted and community-supported network security distributions in the open-source ecosystem.
Here's a dedicted documentation about pfsense firewalls, all home labs and conecpts are diccussed here check out here:
github.com/raajeshmenghwar/working-with-firewalls/pfsense

Why do we want to integrate the pfsense with wazuh?

Pre-requists:

Vmware/VMEXSI Server having
Wazuh-Stack working(manager, dashboard, indexer)
& Pfsense installed.

Step 1: Power up your pfSense VM. Give it a few moments to fully boot up.


Article content
Once it is fully up, you'll see this screen
Article content
Step 2: Next, boot up one of the Linux VMs. For me, I'll choose Ubuntu
Article content
 
Step 3: Boot up the Wazuh VM
 
Article content
Once you see this screen in the Wazuh VM, punch in the credentials shown




Article content
Step 4: Once all of the VMs referenced above are booted up, open a web browser in your Linux VM and type in the Wazuh IP address and login using the username "admin," password "admin". If you don't know the Wazuh IP address. Go to the Wazuh VM and type in "ip a" and press Enter. You'll see the IP address listed. It will look something like the image below:


Article content
 Step 5: Look at your pfSense VM and take note of the IP address assigned to the LAN interface. It will look something like the below image:
 
Article content
 
Once you have the IP address, open a new tab in your browser and type in the pfSense IP and press Enter. You'll see a warning screen popup, similar to the one referenced below
 
Article content
There isn't anything wrong of course. Click "Advanced" and then scroll down and click on "Accept the Risk and Continue."


Once you accept the risk, you'll see the pfSense login screen punch in "admin" for the username and "pfsense" for the password, then press Enter.
 
Article content
 
Step 6: To practice proper security habits, notice on the first screen that you see that there is a warning about the "admin" account password is set to default. Lets click on the "Change the password . . ." link.
 
Article content
 
Once you're on the "Edit User" screen, punch in a new admin password on the "Password" and "Confirm Password" fields.
 
Article content
 
Once you punch in a new password, scroll to the bottom of the page and click on "Save"
Article content
 
Step 7: Next, click on "System" at the top left of the screen
Article content
You'll see a dropdown menu appear. Click on "General Setup."
 
Article content
 Scroll down to the "Timezone" section and click the dropdown menu. Select your time zone. pfSense doesn't automatically detect your time zone upon installation so by setting it here, it will make researching the logs much easier when we come to that step.




Article content
Once you select your time zone, scroll down to the bottom of the page and click "Save."
 
Article content
 Step 8: Click "Status " near the top right of the screen.
 
Article content
 You'll see a dropdown menu appear, click "System Logs" near the bottom of the menu.
Article content
 
Step 9: Click on Settings in the upper right part of the screen.


Article content
 
Under "General Logging Options" at the top of the menu, click the dropdown next to "Log Message Format" and choose "syslog." The reason why is that Wazuh is natively able to ingest and analyze syslog messages easily.




Article content
 (optional) Directly below the "Log Message Format" section, click the checkbox next to "Forward/Reverse Display" that says "Show log entries in reverse order (newest entries on top)." I like to enable this setting because in my opinion it makes researching the logs a little easier. Its up to you though. 
Article content
 
Scroll down to the bottom of the Settings screen and check the box that says "Send log messages to remote syslog server."
 
Article content
 
Directly beneath the checkbox, click the dropdown next to "Source Address." Then click on "LAN" since the data we are examining is going to be within the virtual LAN. 
 
Article content
 
Step 10: Scroll down to where it says "Remote log servers." type in the IP address of your Wazuh server in the format of x.x.x.x:514. See the below image for reference. The reason why we need the ":514" appended to the end of the IP address is that syslog messages are sent over UPD port 514, so this tells pfSense exactly where to send the logs and the port to send the logs through.




Article content
 
Under the "Remote Syslog Contents" section, I strongly recommend that for simplicity sake you only have System Events, Firewall Events, DNS Events and DHCP Events checked. The reason why is that if you select more than that or even "Everything," the amount of logs that you'll have to sort through will be MASSIVE. Its up to you of course.




Article content
 
Once you configure all of the desired settings, scroll down and click on "Save."
 
Step 11: Click "Firewall" at the top middle of the screen.
 
Article content
 
Once you see a dropdown, click on Rules
 
Article content
 
Next, click on the "LAN" section on the upper left portion of the screen
 
Article content
 
Click the green "Add" button with the arrow pointing up to add a new rule to the top of the rules list
 
Article content
 Under the "Edit Firewall Rule" section, click the dropdown next to "Protocol" and click on "UDP." Under the "Source" menu, click the dropdown and select "LAN subnets." Under the Destination menu, click the dropdown next to "Destination and select "Address or Alias." Punch in your Wazuh server IP address in the "Destination Address" field. 


Click the dropdown menus that currently say "(other)" and choose "syslog(514)." Scroll down to the "Extra Options" menu. Look where it says "Description." Type in "Forward syslog entries to Wazuh." If your settings look like the below image, you're all set. 


Article content


Any settings that I didn't mention above you can leave at their default values. Scroll down and click on "Save."
 
Next, we need to switch gears and login to the Wazuh server and basically tell the server to listen for the syslogs that pfSense will be sending to it. By configuring Wazuh to "listen" for the syslog entries, Wazuh will properly ingest/analyze them for us to monitor as part of our lab.
 
Step 12: Open another tab on your web browser and type in your Wazuh server IP address. Login using the admin credentials ("admin" for username, "admin" for password).




Once you're logged in, click the menu button in the top left. Scroll down and click on "Server Management," which will open up a dropdown, then click on "Settings."
 
Article content
Article content
 
You'll be taken to a page with a lot of data and descriptions, but for this exercise, we need to click on "edit configuration" in the upper right corner.
 
Article content
 
You'll be taken to a page of XML code; it is the configuration file of the Wazuh server. enter these new settings into the configuration file:
 
<remote>
 <connection>syslog</connection>
 <port>514</port>
 <protocol>udp</protocol>
 <allowed-ips>x.x.x.x/24</allowed-ips>
 <local_ip>x.x.x.x</local_ip>
</remote>


 (the x.x.x.x/24 above is your subnet IP address, the x.x.x.x under local IP is the IP address of your Wazuh server)
 
Here is what my Wazuh server config file looked like when I finished


Article content
 
Click "Save" in the upper right corner when you're done, then click "Restart Manager," then "confirm."
 
Article content
Step 13: Lets test these new settings easily by logging out of the pfSense Web GUI and then punching in the wrong credentials on purpose to generate a failed login.
 
Article content
Once the failed login happens, lets go back to Wazuh to see if a log showing the failed login appears.  Click the menu button in the upper left corner, then click "explore," then "discover." Notice in the following image that indeed Wazuh is showing a failed login log from syslog. So our configuration settings on both pfSense and Wazuh are set up properly. Nice work!
Article content
 
