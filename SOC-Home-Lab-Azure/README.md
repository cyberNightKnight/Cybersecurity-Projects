# SOC Home Lab 1
#### Written by: cyberNightKnight

## -- Overview --

This lab consisted of setting up a honeypot in Azure and using the virtual machine's logs to build a map based on where the attackers' IP addresses were from.

Additionally, I added an interface that showed the percentage of attacks from each region and an incident generation rule.


**A concept I mention is SIEM. This stands for Security Information and Event Management, and it is a system that analyzes logs in order to detect events.**



### __DISCLAIMER: This lab was mainly developed following a guide created by Josh Madakor, which can be found in the References section. I additionally added a percentage map and an incident generation rule to make the lab more complete.__



## -- Steps --

### 1. Create a Virtual Machine

This VM is the one that will act as the honeypot.
**To do this, an Azure account is needed. In this case the account was created using a free subscription.**

### 2. Create a Resource Group (RG)

_All of this project's resources will be found inside this RG_

Inside the Azure main interface, search for 'Resource Groups' and go to the corresponding interface.

Once in the Resource Groups interface, click on '+ Create' and input the required data. 

For this exercise I used the second location of East US.
  
After the data is complete and correct, click on 'Review + Create'.

!["Resource Groups"]("Progress-Images/Resource_groups.png")

### 3. Create a Virtual Network (VNET)

The VM's public IP--the one attackers will try to connect to--will be created here.

Back in Azure's home page, search for 'Virtual Networks', go to the corresponding interface, and select '+ Create'.

Add the VNet's name and ensure the Region is the same as the RG.

Finally, select 'Review + create'.

__** I left all other configurations as default for 'Security', 'IP addresses', and 'Tags', but it's still useful to check the VNet's configuration **__

!["Created virtual network"]("Progress-Images/VNet_instance.png")

### 4. Create the VM

This VM will be the honeypot.

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
