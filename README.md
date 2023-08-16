# Azure Sentinel (SIEM) - Failed RDP to IP Geolocation Information


<h1>Description</h1>
In this project, I setup Azure Sentinel (SIEM) and connected it to a live virtual machine acting as a honey pot. I then observed live attacks (RDP Brute Force) from all around the world. I used a custom PowerShell script to look up the attackers Geolocation information and plot it on the Azure Sentinel Map!\
<br />




<h2>General Steps of Project</h2>
1. Create Azure Subcription.<br />
2. Create a VM in Azure - turn off external and Windows Firewall off (exposed).<br />
3. Create a log repository in Azure Log Analytics Workspace - ingest logs from the VM.<br />
4. Use PowerShell to IP adrress extraction.<br />
5. Use 3rd party API to derive geo location from IP address.<br />
6. Make a custom log with geo data.<br />
7. Setup Azure Sentinel - create a map to map attack data (countries).<br />

(image of work tree) 

<h2>Environment Used </h2>

- <b>Windows 10 Pro</b> (22H2)



<h2>Utilities Used</h2>

- <b>ipgeolocation.io:</b> IP Address to Geolocation API


<h2>Project walk-through:</h2>
Started by creating a Virtual Machine (VM) for Windows 10 Pro and a new resource group. Make an admin username and password as well. For the NIC security group, 
