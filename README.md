
# Azure VM Honeypot for attack map creation

### Goal
It is a great project for cybersecurity beginners and professionals looking to practice log analysis, threat detection, and SOC operations in a real-world cloud environment.

This project allows us to create a basic home SOC from beginning to end on #Azure, using the free subscription.
We are going to create a virtual machine that is open to the public internet as honeypot, and forwarding logs to a central repository #Azure/loganalyticsworkspace . We then integrate Microsoft Sentinel #SIEM/Sentinel to analyze real-world attack data

Topics covered:

- Creating an Azure subscription and setting up a VM
- Configuring Log Analytics Workspace
- Forwarding logs and integrating with Sentinel
- Querying login attempts and visualizing attack sources #SIEM/Sentinel/KQL

The following is the network architecture that we are trying to create

<img width="661" height="373" alt="image" src="https://github.com/user-attachments/assets/15e0b50b-0619-4386-96f3-5bcbe02bb201" />

Once the Azure account is created, we will then navigate to the website:
https://portal.azure.com

Before getting started with the creation of a honeypot using the azure VM, first we need to discuss few key concepts like 

1. **Resource group** : An Azure resource group is a  container that holds all related resources for an Azure solution. All resources that share the same lifecycle are generally added to the same resource group. In this project, our resource group will be acting as a container to hold both our virtual machine(honeypot) and virtual network together. This allows for easier deployment, updating, and deletion of resources as a group. #azure/resourcegroup

2. **VNet** : This service enables resources like VMs to securely communicate with each other, the internet and on-premises networks. #azure/VNet 

3. **Network Security Group** : A network security group contains [security rules](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview#security-rules) that allow or deny inbound network traffic to, or outbound network traffic from, several types of Azure resources. #azure/NSG

4. **Log Analytics workspace**: An Azure Log Analytics workspace is a centralized data store that allows you to collect, manage, and analyze log data from both Azure and non-Azure resources. It serves as a foundation for Azure Monitor, enabling you to gather data from various applications and resources, whether they are running in Azure, other clouds, or on-premises #Azure/loganalyticsworkspace 

5. **Microsoft Sentinel** : Microsoft Sentinel is a cloud-native security information and event management (SIEM) solution that provides scalable and cost-efficient security across multicloud and multiplatform environments. It integrates AI, automation, and threat intelligence to enhance threat detection, investigation, response, and proactive hunting capabilities. #azure/Sentinel #SIEM/Sentinel 


### Create the Honeypot using Azure virtual machines

Step 1: Create a resource group where we will be installing the virtual machine

Step 2 : Creating a virtual network that which enables us to connect our virtual machine(honeypot) with the internet. This will create a virtual network and a subnet inside that virtual network.

Step 3: Creating a virtual machine, our so called honeypot that is exposed to the internet, so that it can be attacked

Here we can see that under our resource group, post creation of our VM and VNet, we see the resource group hosts the below highlighted information like the VM name, the public IP at which our machine can be reached and the NSG(Could firewall) that it belongs to
<img width="940" height="763" alt="image" src="https://github.com/user-attachments/assets/91f5d3a9-a29e-4ddc-85f6-f1b6695fa89a" />

NSG : It is a cloud firewall in place provided by azure, which will filter traffic basis on the inbound and outbound rules specified within this NSG, which will  determine what traffic can enter from the public network and what can leave from this network to public internet.

In this below screenshot we can see that RDP is allowed from anywhere in the world by anyone to enter this network. 
<img width="875" height="784" alt="image" src="https://github.com/user-attachments/assets/cac2cfae-496b-49b8-a39c-f58d865d2327" />

Since, we want this machine to act as a honeypot and be open to the public, hence, we are creating a new inbound rule stating that anybody can access this machine from the public internet.
<img width="930" height="284" alt="image" src="https://github.com/user-attachments/assets/38c4c1f3-1a36-404c-8bc8-d9d5048edba7" />

Step 4: We will access our virtual machine using the RDP protocol and disable all firewall rules so that the machine can be accessed from anywhere on the internet.
We will use the run command wf.msc, open the windows firewall console, and turn off the "firewall state" for all the three profiles : Domain, Public, Private
<img width="510" height="507" alt="image" src="https://github.com/user-attachments/assets/abfc3c74-1c2a-4eae-90d3-d704a5b787cd" />

Once turned off, the updated rules in the console looks like the following
<img width="381" height="205" alt="image" src="https://github.com/user-attachments/assets/40cabc12-b3af-4b9f-9b0a-0b787e892a29" />

Step 5: Here we are using our local host to see if we able to connect to our virtual machine that we have just opened to the entire internet.
<img width="861" height="407" alt="image" src="https://github.com/user-attachments/assets/33ef1833-35a8-401a-9307-3cef8fde5744" />

This is how architecture looks and below is a simple recap:
- We have our tenants, in our active azure subscription. We've created our resource group in which we have crated our virtual network and subnet. 
- Then we have our virtual machine attached to our subnet. Our VM is enabled with a public IP address and an NSG attached to it where we have created our custom inbound rule to allow all traffic from anywhere in the world.
- Then we have disabled the windows firewall inside our virtual machine. 
<img width="628" height="335" alt="image" src="https://github.com/user-attachments/assets/b489af17-5677-4411-94e0-b4d4e4d9a44f" />

### Logging into the VM and inspecting logs

Login to your virtual machine
Open up Event Viewer and inspect the security logs and see if there are any failed attempts
<img width="820" height="794" alt="image" src="https://github.com/user-attachments/assets/89864cb5-68cf-4796-aebd-5ff61596f802" />

### Log Forwarding and KQL

Step 1: Create a log analytics workspace that will create our log repository 

Step 2: Then will link our log analytics workspace that we've created earlier, to Microsoft Sentinel (SIEM) that can accessed through sentinel to see the logs from our log repository.

Step3: We will install a connector from the content hub, under content management. This will connect our virtual machine to the log analytics workspace using the connector "Azure Monitoring Agent" #azure/AMA via "Windows security event"
<img width="903" height="869" alt="image" src="https://github.com/user-attachments/assets/f6a9678a-9f60-4b0e-b527-1c2403e2e318" />

Step 4: We will create a data collection rule for our virtual machine under the connector page
<img width="875" height="542" alt="image" src="https://github.com/user-attachments/assets/383f4ab8-b618-45fb-8407-e7d2b8de4ec4" />

Step 5: Post connection, we can see that the VM and the log analytics are now connected through the AMA agent that we installed earlier and the same can viewed inside the VM under the Settings > Extensions
![[Pasted image 20260416165757.png]]

Step 6: Query for logs within the LAW
We can now query the Log analytics workspace as well as the SIEM, sentinel directly, which we will do soon
Note: Querying logs in here is a really important skill that you MUST have if you want to work in security operations. Depending on where you work, you need to know SQL, KQL, or SPL, but these are all basically the same thing.

Observe some of your VM logs:

```kql
SecurityEvent
| project TimeGenerated, Account, Computer, EventID, Activity, IpAddress
```

<img width="1142" height="669" alt="image" src="https://github.com/user-attachments/assets/78d54763-3268-4ed4-8fef-37f9a2ab70f2" />

### Log Enrichment and Finding Location Data - Creating Watchlist

Observe the SecurityEvent logs in the Log Analytics Workspace; there is no location data, only IP address, which we can use to derive the location data.
We are going to import a spreadsheet (as a “Sentinel Watchlist”) which contains geographic information for each block of IP addresses.

Step 1: Within Sentinel, create the watchlist:

Step 2: Allow the watchlist to fully import, there should be a total of roughly 54,000 rows.
<img width="623" height="770" alt="image" src="https://github.com/user-attachments/assets/d4d28898-13e1-4970-9c17-1b74b55d7234" />

Step 3: This is our watch list that we just uploaded into Sentinel/Defender, so you can see where the attacks are coming from.
<img width="1241" height="572" alt="image" src="https://github.com/user-attachments/assets/d16b03fe-fae2-43d6-995c-31e26042ceb9" />





