---
layout: default
title: Wazuh and Active Directory Lab
permalink: /projects/Wazuh-and-Active-Directory-Lab/
---
# Wazuh and Active Directory Lab

lab focused on Windows Active Directory administration, security monitoring, attack simulation, and detection engineering fundamentals.

## Environment used:
* Windows Server 2025 Domain Controller
* Windows Server 2025 file server
* Windows 10 as domain-joined workstation x3
* same as always Kali Linux as attacker machine
* Ubuntu server as Wazuh host
* Virtualbox with internal network


Starting where I left, we have domain with 3 workstations, domain controller and file server.

To make thing begin to look like a somewhere real Security Operations Center I installed Wazuh on ubuntu server 

After a few hours of unexpected error with network interface cards and Virtualbox overall I saw on my monitor what I wanted to see:

![active](SecurityProjects/images/Wazuh-and-Active-Directory-Lab/wazuh-active.png)

![active](/SecurityProjects/images/Wazuh-and-Active-Directory-Lab/wazuh-agenty.png)


Installing Wazuh agents turned out to be more complicated than I could have imagined as all my workstations messed up network cards somehow - it took a while to clean them up

![active](,,/images/Wazuh-and-Active-Directory-Lab/wazuh-agenty3.png)


Then I also installed `sysmon` on them

`sysmon` - popular utility to monitor and deeply log system activity to the Windows Event Log

Now I wanted to test logs collecting and took for example new user creation (EventID- 4720)
Despite creating new users in the domain, no alerts related to account creation were appearing in Wazuh.
Searching for Event ID 4720  returned no results.
Additionally, checking the Windows Security log locally with:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4720} -MaxEvents 5 | Select TimeCreated,Id,Message
```

![active](,,/images/Wazuh-and-Active-Directory-Lab/4720-1.png)

returned only old events - The first step was to verify that the Wazuh agent was correctly collecting Security log events:

```xml
<localfile>
<location>Security</location>
<log_format>eventchannel</log_format>
</localfile>
```
it was on its place so I went further - Windows audit policy:

```powershell
auditpol /get /subcategory:"User Account Management"
```
bingo: No Auditing

![active](,,/images/Wazuh-and-Active-Directory-Lab/auditpol.png)

I enabled auditing for the User Account Management subcategory:

![active](,,/images/Wazuh-and-Active-Directory-Lab/auditpol2.png)

This time, a new Event ID 4720 entry appeared with today's date:

![active](,,/images/Wazuh-and-Active-Directory-Lab/auditpol3.png)

 and the event was successfully ingested by Wazuh and could be located using the following Threat Hunting query:

 ```bash
data.win.system.eventID:4720
```
![active](,,/images/Wazuh-and-Active-Directory-Lab/auditpol5.png)

`rule.id 60109` - Account enabled or created


next I decided to create my own Wazuh rule... and it turned out into long troubleshooting.
Long story short: rule seemed to be okay, but didn't generate any alerts.
I didn't have patience to document everything, but here is quick sum up:


I created simple rule to detect `net user` usage in command line


```xml
<rule id="100100" level="8">

    <if_sid>92031</if_sid>

    <field name="data.win.eventdata.commandLine" type="pcre2">
        (?i).*net1?\s+user.*
    </field>

    <description>Possible Account Discovery using net user</description>

    <mitre>
        <id>T1087</id>
    </mitre>

    <group>
        attack.discovery,
        account_discovery,
        custom_detection,
    </group>

</rule>
```


wazuh engine syntax validation of course went smoothly, no error after command:

![active](,,/images/Wazuh-and-Active-Directory-Lab/test_kompilacji_parsera.png)\


I checked in EventViewer if Sysmon events reached Wazuh

   and of course they did
   

I verified event ingestion

   I confirmed that Wazuh Manager was receiving the events by inspecting archived events.
   
   ```bash
   sudo grep -i "net.exe" /var/ossec/logs/archives/archives.json | tail -1
   ```
   
![active](,,/images/Wazuh-and-Active-Directory-Lab/event_ingestion.png)


then were few attempts to check if sid 92031 is correct in `/var/ossec/ruleset/rules` directory, it was there



finally after multiple more syntax attempts  I removed the example SSH rule included in `local_rules.xml.`

![active](,,/images/Wazuh-and-Active-Directory-Lab/basic2.png)

 <br>
 
Guess what happened?

<br>




![active](,,/images/Wazuh-and-Active-Directory-Lab/worked.png)

IT WORKED


So, it should not help, but it did. Maybe problem with XML formatting, maybe with something else, I dont know.

For the future I will always keep local_rules.xml clean from  unused example rules before building custom detections.

Moreover, the story didn't end here, as it turned out it was only a begining of a debugging story, complete is over here:

[Technical Case Study](https://github.com/Aleksander9191/SecurityProjects/blob/main/projects/Wazuh%20SIEM%20Complete%20Debugging%20Case%20Study%20and%20Troubleshooting%20Process.md)

and in github issue:
[here](https://github.com/wazuh/wazuh/issues/37589)





