# Azure-SIEM-Lab
Azure SIEM Lab

Building my own SOC (Security Operations Center) by deploying SIEM (Security Information and Event Manager) which monitors and generates alerts for all the devices in the lab.



First we are going to set up our first virtual machine (VM), i like to go for the preset configurations because it is faster altho if I need something specific I will go with custom configuration.


We need to create a new resource group. This is the container for all the things we will be creating.

The image type is going to be Windows 11, but we can also set up a Windows server, we can set up and AD or a hybrid environment (hybrid environments are joined to the cloud, in this caser in Entra), we can have a Linux device deployed as well which is used for a threat intelligence feed.

We are going to put a Username and Password.
We are going to be monitoring RDP (Remote Desktop Protocol) Events, because we can get traffic from the rest of the world very quickly to an open RDP Port. So we are going to allow RDP connections.

Azure will create an IP address and a Public IP, we can live everything else default.

We can set up Microsoft signing in with an Entra ID.

Once everything is ready we can press “review and create” and our VM will be deployed.

We can connect to our VM via RDP with the credentials we created.
We do not need to set up anything on the device. If we check In the network setting we can see that it is publicly facing and we have RDP open on a TCP port.


SIEM setup
While the VM is being deployed we can deploy Sentinel which is Microsoft’s SIEM tool. To do that we need to create a new Log analytics workspace, attach it to the resource group and we need to make sure we set it to the same region as the resource group, this will minimize the delay in the event logging.

Again we select “review and create” and the log analytics workspace is deployed.

Now once everything is created we are going to add Microsoft Sentinel to our Log analytics Workspace, so they are connected and talking to each other.


If we click on “Overview” we can see the dashboard of the Sentinel SIEM. It shows all the information like Incidents, Automations, Data, Analytics, we can add different tiles that show different information.



Now that Sentinel is up and running we must go back to our Log analytics Workspace and add the VM. We have to add its event logs and send it to our log analytics workspace which is then going to send it to the Sentinel instance.

In the workspace, if we go to agents we can see that we have 0 computers connected. So all the while the VM is actively logging these events, like any windows computer would, we can't see those events. We have to pull them from the endpoint and send them to Sentinel.



To do that we have to set up a Data Connector. If we click into the content hub which contains all the known established connectors, we can install AMA which is Azure Monitor Agent



Now if we refresh our Data connectors page we will be shown two connectors, because AMA supports two different event ingestion agents.



Now we need to go into the newer AMA agent and set up a data collection rule. We need to select our VM and collect All security events.





Now if we go into Microsoft Sentinel’s Logs and run “SecurityEvent” Query we will see all logged security events on our VM

(Most of the events will be brought force attacks)
What we are interested in is creating a rule that checks for successful sign-ins via RDP.
For that we need to simply add “| where Activity contains "success"” and we will see all of the successful sing-ins. They are going to be System accounts that are signing in.



If we want to generate an alert that is not a system account or rather an RDP connection, we can add “and Account !contains "system"”. If we run this query no results will be found because there were only system account singing in.



Now we can create a new Sentinel alert. We can set the severity to High and the MITRE ATT&CK to Initial Access because this is initial access on our VM for VRDP in this case.
Next we can test our query again to see if it works, we are going to set it to run every five minutes, more of a real time detection, it will start running immediately. The alert threshold is how often it creates an alert, we are going to set it to 0 so after the event occurs one time it will generate an alert. We are also going to group all of the events into a single alert.
We are also going to create incidents from alerts triggered by this rule.



If we go back to the Analytics page where all of our rules are, we can see the newly created rule and if we connect to the VM via RDP an event will be generated and an incident will be generated from the event.


