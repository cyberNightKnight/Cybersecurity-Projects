# SOC Home Lab 1
#### Written by: cyberNightKnight

## -- Overview --

This lab consisted of setting up a honeypot in Azure and using the virtual machine's logs to build a map based on where the attackers' IP addresses were from.

Additionally, I added an interface that showed the percentage of attacks from each region and an incident generation rule.


**A concept I mention is SIEM. This stands for Security Information and Event Management, and it is a system that analyzes logs in order to detect events.**



### __DISCLAIMER: This lab was mainly developed following a guide created by Josh Madakor, which can be found in the References section. I additionally added a percentage map and an incident generation rule to make the lab more complete.__



## -- Steps --

### 1. Create a Virtual Machine

_This VM is the one that will act as the honeypot._
**To do this, an Azure account is needed. In this case the account was created using a free subscription.**

### 2. Create a Resource Group (RG)

_All of this project's resources will be found inside this RG_

Inside the Azure main interface, search for 'Resource Groups' and go to the corresponding interface.

Once in the Resource Groups interface, click on '+ Create' and input the required data. 

For this exercise I used the second location of East US.
  
After the data is complete and correct, click on 'Review + Create'.

!["Resource Groups"]("Progress-Images/Resource_groups.png")

### 3. Create a Virtual Network (VNET)

_The VM's public IP--the one attackers will try to connect to--will be created here._

Back in Azure's home page, search for 'Virtual Networks', go to the corresponding interface, and select '+ Create'.

Add the VNet's name and ensure the Region is the same as the RG.

Finally, select 'Review + create'.

__** I left all other configurations as default for 'Security', 'IP addresses', and 'Tags', but it's still useful to check the VNet's configuration **__

!["Created virtual network"]("Progress-Images/VNet_instance.png")

### 4. Create the VM

_This VM will be the honeypot._

Just like in the previous steps, go to the 'Virtual machines' interface and select '+ Create' > 'Virtual machine.'

Add the RG and name the VM.

__** It's not recommended for the VM to be called in a similar way as the rest of the Resource Group's resources because the attackers might see it. So, I called it something similar to what a business would call it ('ADMON-NET-WEST-2') **__

As for the rest of the configuration:
  - Availability: No redundancy required
  - Security Type: Standard
  - Image: Windows 10 Enterprise, version 22H2 - x64 Gen2(free services eligible)
  - Size: Standard_D2s_v3
  - Set the username and password
  - Check licensing agreement

In the Disk section:
  - I selected standard HDD for lower cost, but it can be any other disk

In the Network section: 
  - Check Delete public IP and NC when VM is delete

Skip Management Configuration.

Finally, in the Monitoring section:
  - Disable Boot diagnostics

After verifying all data is correct, select 'Review + create'

!["Created virtual machine"]("Progress-Images/VM-config.png" alt="Created virtual machine")

### 5. Add NSG (firewall) rule to allow ANY traffic in Azure

_The purpose of this is ensuring that the VM is detected as vulnerable and thus is seen as a viable target for attacks._

In the Resource Groups page look for the resource that has the .nsg extension.

__** For inbound rules, only the Remote Desktop Protocol (RDP) is initially allowed**__

Delete the inbound rule with 300 - RDP, then go to 'Settings' > 'Inbound security rules' > '+ ADD'.

Change 'Destination Port' to '*' (ANY) and name the rule and select 'ADD'.

!["Created firewall inbound rule"]("Progress-Images/firewall_inbound_rule.png")

### 6. Disable internal Windows firewall

_This it to ensure the VM is as vulnerable as possible. This firewall is also the one where the logs will come from._

In the 'Virtual machines' interface select the VM created for this project.

> Connect remotely to the VM.
__** Since I use a Mac (iOS), I had to use an app called 'Windows App' to connect to the Virtual Windows Machine, but for devices with Windows OS this can be done using the 'Remote Desktop Connection (RDP)' app. **__

In the 'Windows App' application, select 'Add PC' and put the VM's public IP as the name.

Once inside the VM, look for 'wf.msc' (Windows Defender), select 'Windows Defender Firewall Properties', and Turn all firewall 'Profile' states off > 'Apply' > 'Ok'.

_When pinging the public IP, the VM is now reachable._

__** Note: When trying to access with incorrect credentials, the logs can be seen in 'Event Viewer' > 'Security'.**__

### 7. Forward logs to Azure

_In order to generate the map, percentage diagram, and incident rules, the logs must be obtained from the VM's firewall._

  - Create Log analytics workspace (LAW)
    _LAW will receive the logs from the VM, but to do this they must be accessed through Sentinel._
    > 'Log Analytics Workspaces' interface > '+ Create' > Fill required parameters
    
    > 'Microsoft Sentinel' interface (SIEM) > Create a link to the Log analytics workspace
    __** Now the log workspace will be linked to the SIEM **__

  - To connect the VM and LAW through Sentinel:
    > 'Content Management' > 'Content Hub' > 'Install Windows Security Events' > 'Manage Windows Security Events' > Check 'WSE via AMA' (Azure Monitoring Agent)
    > Open the Connector Page and create a data collection rule to forward the logs and grant them access to the SIEM
    > 'Resources' interface > VM > 'Create'
    __** Simultaneously, it's possible to see in the VM interface that the Connector is being installed. **__

!["Sentinel instance"]("Progress-Images/Sentinel-Instance.png")

#### To see logs:####
  > 'Logs Analytics Workspace' > 'Logs' > 'Tables' > search 'SecurityEvent'

To add more specific parameters, set it to KQL mode.
  __** To only see certain parameters, use '| project {list of params}', or to only get a certain user, '| where Account == '{account name}'' **__
  __** To search the location of an IP address: https://ipinfo.io/{ip address} **__
  __** To link IP address and its origin country, upload the .csv file with IP-location equivalences to Azure. I obtained it from the video's description (check the References section of this page) **__

In Sentinel:  Sentinel instance > 'Configuration' > 'Watchlist'.
  _It will redirect to Windows Defender._There, 'Microsoft Sentinel' > 'Configuration' > 'Watchlist' > '+ New'.

  - Name & Alias: in this case it was 'geoip', but it can be any name
  - Source: Drop the file; Search key: 'network'
  - Review + Create
__** To see watchlist from LAW, go back to KQL mode and input <i>'_GetWatchlist("watchlist name")'. Then run it **__

### 8. Create the map

_This will help visualize where the attacks are coming from and potentially block IP addresses from these areas._
_The more attacks originate from certain region, the bigger the corresponding circle in the map will be. _

In Sentinel (which will redirect to Defender), select 'Threat management' > 'Workbooks' > 'Add workbook' > 'Edit'

Remove all of the elements, select 'Add query'/'Add data source + visualization' and go to 'Advanced editor'.

In the advanced editor I pasted the data contained in the map.json file that cand be found in the video's description.

Select 'Done editing' (at the top)

!["Created instance of Windows Defender"]("Progress-Images/Defender_instance.png")
!["Created map in its initial state"]("Progress-Images/initial_map2.png")
_This is the map after a couple hours being active._

### 9. Create percentages diagram

_I added this to get a clearer view of where most of the attacks were coming from. We can measure this with plain numbers, but I find percentages easier to deal with, mainly for statistical reasons._

In this case I created a diagram with the percentage of attacks per country.

To do this, I followed the same initial steps as the map, except I selected 'Pie chart' as the type.

The input in the advanced editor also changed:

```let IPGeoMap = _GetWatchlist("geoip");```
``` let TotalEvents = toscalar( SecurityEvent```
```| where EventID == 4625 ```
      ```| count ); SecurityEvent ```
      ```| where EventID == 4625 
      | evaluate ipv4_lookup(IPGeoMap, IpAddress, network) 
      | summarize FailureCount = count() by countryname 
      | extend Percentage = round(todouble(FailureCount) / todouble(TotalEvents) * 100, 2) 
      | project country = tostring(countryname), Percentage
      | order by Percentage desc```

_This essentially generates a Pie chart based on the amount of failed login attempt per country. The IPs and locations are resolved with the ipv4_lookup function. The output is then converted to a percentage_

!["Created percentage chart"](Progress-Images/initial_map.png)

### 10. Generate incidents
_Incidents are an essential part of a SOC because they notify when an anomaly is present in the system. The sooner these anomalies are detected, the sooner the risk can be mitigated. In this case, the incidents only focus on persistent access attempts, but in a real SOC, these would cover a larger range of events._

To generate incidents, I went back to 'Sentinel' > 'Configuration' > 'Analytics' > 'Create Scheduled Query Rule'

In this case it would only generate an incient if there was a failed login attempt, focusing on persistence attacks.

Here's how the rule ended up looking:
!["Defender rule name"](Progress-Images/Defender_RuleName.png)
!["Full Defender rule"](Progress-Images/Defender_rule_full.png)

## -- Results --

Within the first few hours there were already many attacks.

!["Initial attack map"](Progress-Images/initial_map2.png)
!["Initial percentage"](Progress-Images/initial_map.png)

After leaving the VM running for more than 8 hours, these were the results:
!["Final attack map"](Progress-Images/last_map.png)
!["Final percentage"](Progress-Images/last_percentage.png)

__** Take into account the incidents are grouped and each of these may contain up to 150 similar incidents **__
!["Last incidents"](Progress-Images/last_grouped_incidents.png)

## -- Lessons learned --

The purpose of this lab was to learn about the deployment of honeypots, how we can detect and identify attackers, and how to map those attackers based on the IP's location.

Additionally, I also learned how to generate incidents and turn log data into percentages.

However, I'd say perhaps the most relevant thing I learned is how little time it takes for an attacker to find a vulnerable device; within the first two hours there were at least 2 access attempts already. After 8 hours there were more than 60k.

This is why cybersecurity is very important in this technological era: to help protect people from those who are actively looking to attack.

### Thank you !

## -- References --

> 'Cyber Home Lab from ZERO and Catch Attackers! Free, Easy, and REAL (Microsoft Sentinel 2025)' by Josh Madakor - https://www.youtube.com/watch?v=g5JL2RIbThM

> 'Create Incidents from alerts' by Microsoft: https://learn.microsoft.com/en-us/azure/sentinel/create-incidents-from-alerts
