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
7. Setup Azure Sentinel - create a map to map attack data (countries).<br /><br />
<p align="center">
<img height="80%" width="80%" src= "https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/1aa08546-f244-4c11-b224-9a00f406d0bd">

<br />
<h2>Environment Used </h2>
- <b>Windows 10 Pro</b> (22H2)

<h2>Utilities Used</h2>
- <b>ipgeolocation.io:</b> IP Address to Geolocation API


<h2>Project walk-through:</h2>
Start by creating a Virtual Machine (VM) for Windows 10 Pro and a new resource group. Make an admin username and password as well. For the NIC security group, create a new firewall by create our own custom inbound rule that allows everything into the VM (TCP pings/ ICMP pings etc), as we want it exposed to worldwide attackers and no traffic droppage. Set the destination port ranges to '*' and set the priority low, to around '100'. Then 'Review and Create'. <br /><br />

<p align="center">
<img height="80%" width="80%" src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/d56bfb9f-6a65-4db3-9a94-81353380d53b">

<br />
<br />

Next, head over to the 'Log Analytics Workspace' in order to ingest logs (Windows Event logs) from the VM. Select the resource group we just made ('honeypot-vm'), and after 'Review and Create'. Next, head to 'Microsoft Denfender for Cloud'and open the 'law-honeypot' that we just created under 'Enviroment Settings'. Turn on 'Servers' and 'Foundational CSPM'. Save it. <br /><br />

<p align="center">
<img height="80%" width="80%" src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/f04cecaf-0907-4548-a8ac-fc9d0d3ca870">

<br /><br />

Head to the next option under called 'Data Events' and choose 'All Events' and 'save'. Then head to our 'Log Analytics Workspace' and connect the VM to it. In the meantime that is working, head on over to 'Microsoft Sentinel' to set it up. Click 'Create' and choose the workspace we just made:
<br /><br />

<p align="center">
<img height="80%" width="80%" src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/5616502b-ee67-4339-b535-149ef065475f">
<br /><br />
         
Once the VM is has started running, click on it then under overview, copy the 'Public IP address':
<br /><br />
<p align="center">
<img height="80%" width="80%"  src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/6db1b455-3e28-4796-a0b0-79b2a08286e3">
<br /><br />

Then open 'Remote Desktop Connection' on your own computer and pase the address in as well as the admin username and password that you had created earlier on:
<br /><br />
<p align="center">
<img height="80%" width="80%" src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/426ce61b-afc2-40a6-a22d-ca95a6d203ae">
<br /><br />


Once you have opened the VM, go to 'Event Viewer' inside the VM. Then head over to 'Windows Logs' then 'Security' and wait for it all to load. It displays all security on this VM. Focus on <b> Event ID: 4625 </b>, as it shows us the failure logs throug RDP. If we open one of them, it tells us the IP address and rthe name of the workstation that tried to login and failed, and this (IP addresses_ is what we will start using to ge their geolocation. 
<br /><br />
<p align="center">
<img height="80%" width="80%" src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/e097483a-4756-4655-b412-3130f3214d88">
<br /><br />

We then head on over to our computer and ping the VM IP address using 'ping (IP address) -t' using the CMD to see if the VM's firewall is working or not. Initially it says 'Request timed out', so now we go to the VM's firewall settings and turn the domain, private and public firewalls off. Then we ping it again and see the echo requests this time round:
<br /><br />
<p align="center">
<img height="80%" width="80%" src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/87f8108c-db88-4a91-bb09-6b3a332b332b">
<br /><br />

We then download a PowerShell Script (attached to this repo) and run Windows Powershell ISE and then create  a new script and paste the script we just downloaded. At the same time, sign in to the 'ipgeolocation.io' website and come to your main dashboard page. It will give you a custom API key and copy that into the script in PowerShell ISE, and then run it:
<br /><br />
<p align="center">
<img height="80%" width="80%" src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/45987ceb-2354-4964-82df-1b9c91403e94">
<br /><br />
The script looks through the event security log we saw ealier and grabs all the events and the IP address of the people who failed to login. It then grabs their corresponding geo data and creates a new log file called 'failed_rdp'. The purple output we see are the 4625 failed log ons and sends it to the IP address API and gets the geo data.
<br /><br />
<p align="center">
<img height="80%" width="80%"src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/924b57c6-1b3c-4524-95e9-8789b0c9818c">
<br /><br />

Now on our computer, we create a custom log inside analytics workspace that allows us to bring the geo data log into our custom workspace. We go to 'Tables' in our Log Analytics Workspace and create a new MMA-based log. For the sample log, we copy the contents of failed_rdp log file from the VM to our computer, and is used to train Log Analytics what to lok for in the log file. Type in the correct Windows Path. Then, head over to 'Logs' then make a new Sentinel Map Query and type in the folowing code:

```ruby
FAILED_RDP_WITH_GEO_CL 
| extend username = extract(@"username:([^,]+)", 1, RawData),
         timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
         latitude = extract(@"latitude:([^,]+)", 1, RawData),
         longitude = extract(@"longitude:([^,]+)", 1, RawData),
         sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
         state = extract(@"state:([^,]+)", 1, RawData),
         label = extract(@"label:([^,]+)", 1, RawData),
         destination = extract(@"destinationhost:([^,]+)", 1, RawData),
         country = extract(@"country:([^,]+)", 1, RawData)
| where destination != "samplehost"
| where sourcehost != ""
| summarize event_count=count() by latitude, longitude, sourcehost, label, destination, country

```
Upon running it, we get the failed RDP logs. We then want to extract geodata fields like lattitude, longitude etc. To do this, we can use the code:

```ruby
Failed_RDP_Geolocation_CL
| parse RawData with * "latitude:" Latitude ",longitude:" Longitude ",destinationhost:" DestinationHost ",username:" Username ",sourcehost:" Sourcehost ",state:" State ", country:" Country ",label:" Label ",timestamp:" Timestamp
| project Latitude, Longitude, DestinationHost, Username, Sourcehost, State, Country, Label, Timestamp
| where destinationhost_CF != "samplehost"
| where sourcehost_CF != ""
```
Next, for visualization choose 'Map' and size 'Full' for example. In the Map Settings, set Latitute to latitude and Longitude to longitude, and for the 'Metric Label', choose 'Label' and for 'Metric Value', choose 'Event Count'. Then Apply and see the map. This map represents the location of failed live cyber attacks from around the world!!
<br /><br />
<p align="center">
<img height="80%" width="80%"src="https://github.com/aashah23/AzureSentinelSIEM/assets/66967848/cc963ff6-9c41-47ff-bf41-f8c91853fb2d">
<br /><br />


What we see is that after around 30 minutes of running the VM, there were slightly over 250 RDP attacks on it, with most coming from Netherlands in this case.
