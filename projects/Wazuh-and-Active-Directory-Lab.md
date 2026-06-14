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

![active](/SecurityProjects/images/Wazuh-and-Active-Directory-Lab/wazuh-active.png)

![active](/SecurityProjectsimages/Wazuh-and-Active-Directory-Lab/wazuh-agenty3.png)

Installing Wazuh agents turned out to be more complicated than I could have imagined as all my workstations messed up network cards somehow - it took a while to clean them up

![active](/SecurityProjectsimages/Wazuh-and-Active-Directory-Lab/wazuh-agenty.png)

Then I also installed sysmon on them

sysmon - popular utility to monitor and deeply log system activity to the Windows Event Log

