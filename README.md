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
Start by creating a Virtual Machine (VM) for Windows 10 Pro and a new resource group. Make an admin username and password as well. For the NIC security group, create a new firewall by create our own custom inbound rule that allows everything into the VM (TCP pings/ ICMP pings etc), as we want it exposed to worldwide attackers and no traffic droppage. Set the destination port ranges to '*' and set the priority low, to around '100'. Then 'Review and Create'. <br />

(insert Image 2)
<br />

Next, head over to the 'Log Analytics Workspace' in order to ingest logs (Windows Event logs) from the VM. Select the resource group we just made ('honeypot-vm'), and after 'Review and Create'. Next, head to 'Microsoft Denfender for Cloud'and open the 'law-honeypot' that we just created under 'Enviroment Settings'. Turn on 'Servers' and 'Foundational CSPM'. Save it.
(insert Image 3)
<br />

Head to the next option under called 'Data Events' and choose 'All Events' and 'save'. Then head to our 'Log Analytics Workspace' and connect the VM to it. In the meantime that is working, head on over to 'Microsoft Sentinel' to set it up. Click 'Create' and choose the workspace we just made:

(insert Image 4)
<br />
Once the VM is has started running, click on it then under overview, copy the 'Public IP address':

(insert Image 5)
<br />

Then open 'Remote Desktop Connection' on your own computer and pase the address in as well as the admin username and password that you had created earlier on:

(insert Image 6)
<br />


Once you have opened the VM, go to 'Event Viewer' inside the VM. Then head over to 'Windows Logs' then 'Security' and wait for it all to load. It displays all security on this VM. Focus on <b> Event ID: 4625 </b>, as it shows us the failure logs throug RDP. If we open one of them, it tells us the IP address and rthe name of the workstation that tried to login and failed, and this (IP addresses_ is what we will start using to ge their geolocation. 

(insert Image 7)
<br />

We then head on over to our computer and ping the VM IP address using 'ping (IP address) -t' using the CMD to see if the VM's firewall is working or not. Initially it says 'Request timed out', so now we go to the VM's firewall settings and turn the domain, private and public firewalls off. Then we ping it again and see the echo requests this time round:

(insert Image 8)
<br />

We then download a PowerShell Script (attached to this repo) and run Windows Powershell ISE and then create  a new script and paste the script we just downloaded. At the same time, sign in to the 'ipgeolocation.io' website and come to your main dashboard page. It will give you a custom API key and copy that into the script in PowerShell ISE, and then run it:

(insert Image 9)
<br />
The script looks through the event security log we saw ealier and grabs all the events and the IP address of the people who failed to login. It then grabs their corresponding geo data and creates a new log file called 'failed_rdp'. The purple output we see are the 4625 failed log ons and sends it to the IP address API and gets the geo data.

(insert purple)
<br />

Now, we create a custom log inside analytics workspace that allows us to bring the geo data log into our custom workspace. 
