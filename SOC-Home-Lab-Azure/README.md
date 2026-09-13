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
**To do this, we first need to have an Azure account. In this case the account was created using a free subscription.**

### 2. Create a Resource Group

