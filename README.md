<h1>Wazuh its Creation and Integration with Active Directory</h1>
General Purpose: After creating the AD and having some systems and accounts in it I wanted to add more features to further develop my IT, Cybersecurity, Management skill set. A SIEM is a nessary part of any infrastructure to monitor security events across all amachines in the environment. Wazuh was a natural choice since it is open-source, provides FIM and integrates well with Windows. 

<h2> Prerequisites </h2>

  - Active Directory Environment
    - Windows Server 2019 DC already configured (`mydomain.com`), with DNS, DHCP, and NAT.
    - Windows client joined to the domain.
  - Wazuh Server VM 
    - A separate Linux-based VM (Debian) or Windows Server for installing Wazuh.
       - I had a VM of Wazuh load it separately so it was accessed as a terminal. In hindsight it is better to set it up under a VM or Win Server because when updating VBox it would break the Wazuh VM kernel.
     - Configuring network on the Wazuh VM to communicate with your domain controller and any domain-joined workstations.
  - Sysmon Tools (For advanced Windows Logging)
    - Download the program from Microsoft Sysinternals.
    - Optional custom config file for Sysmon to filter events.

<h2> Wazuh Setup on a Dedicated VM (This is what I had setup) </h2>

  - Creating the Wazuh VM
      - Research how much information your wanting to traffic logs, files, connections etc. Configure Accordingly
      - Configure the network setting to stay inside the AD env. (e.g., 172.20.0.x)
    - Login and Update the OS (Since its on Linux, Update often and have it updated before installing SIEM components.) 
      `wazuh-user / wazuh`
      `sudo -i`
      `sudo yum update`
  - Accessing the Dashboard
    - Once installed, go to:
      `https://<wazuh-server-ip>:5601` (Default login will be admin/admin) 
      
<h2> Connecting Active Directory Hosts to Wazuh </h2>

  - Install the Wazuh Agent on the Domain Controller
    - Obtain the Agent Installer
      - Windows Server 2019 (Domain Controller), download the Wazuh agent `.msi` from Wazuh website
      - Use the Dashboard agent configuration.
    - Install and Configure
      - Open`.msi`, or install via command line (PowerShell). Provide the Wazuh Manager IP (`172.20.0.x`) when prompted.
        `msiexec /i C:\path\to\wazuh-agent.msi /quiet WAZUH_MANAGER="172.20.0.x"`
    - Verify Connectivity
      - Wazuh Dashboard → Agents, confirm the DC agent is listed and Online.
  - Install the Wazuh Agent on Windows 10 Clients
    - Same Steps as with the Domain Controller.
    - Check Dashboard and that new agents are Active.
   
<h2> File Integrity Monitoring (FIM) in an AD </h2>

  - Edit `ossec.conf` on EACH Agent
    `<directories whodata="yes" report_changes="yes" realtime="yes">C:\Users</directories>`
  (This will enable Wazuh to monitor and log ANY change within \Users. Optionally you can set the Frequency to a different time by seconds.)
  - Restart the agent to apply changes: PS
    `net stop Wazuh`
    `net start Wazuh`
  - Configuring FIM consider monitoring critical AD folder like `C:\Windows\SYSVOL` or `C:\Windows\NTDS` to realtime since it handles policy or user directory changes.

<h2> Sysmon Integration for Advanced Windows Logging </h2>

  - Installing Sysmon Basic and Custom Config (recommended) 
    - PS: `.\Sysmon64.exe -i -accepteula`
    - PS: `.\Sysmon64.exe -i -accepteula -c C:\tools\sysmon\sys_made_config.xml`
      - Creating a separate config file (sys_made_config.xml) allows the user to get specific settings, the basic config is a lot of processes and can bog down the system it is watching with unneeded notifications and flags. So use this to filter out noisy events and focus on critical process/network logs.
   - RENAME SYSMON SERVICES (Obfuscation) 
      `.\Sysmon64.exe -i -d <new_driver_name>`
      `Copy-Item .\Sysmon64.exe .\<new_service_name>.exe`
      `.\<new_service_name>.exe -i -d <new_driver_name>` 
  - It is important to change the name of the config file because it will slow down an attacker from simply shutting down a service that alerts the system admin of a breach. It can still be detected by altitude IDs and Windows Event Logs but hopefully give the admin enough time to catch on.
  - Keep driver/service names under 8 characters that's the max.
  - Check `fltMC.exe` to avoid altitude conflicts (e.g., two services running 90000). 
  - Removing Sysmon
    - Incase it breaks and needs a clean file.
      `sysmon.exe -u force`
      
<h2> Common Wazuh Management Tasks </h2>

  - Removing Agents
    - List Existing Agents:
      `sudo /var/ossec/bin/manage_agents`
    - Remove an Agent by ID:
      `manage_agents -r <agent_ID>`
      `systemctl restart wazuh-manager`
  - Restart Services: (Always restart after configuring ossec.conf)
    - General Command:
      `systemctl restart wazuh-<service>`
    - Examples:
      `wazuh-manager`
      `wazuh-dashboard`
      `wazuh-indexer`
  - Dashboard Not Loading:
      `systemctl status wazuh-indexer`
  Next Steps
    Enable Alerting in Wazuh to notify of critical security events via email or Slack.
    Automate account provisioning or removal in AD, then track events in Dashboard











   
