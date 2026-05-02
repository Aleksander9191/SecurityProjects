---
layout: default
title: Enterprise Active Directory Cybersecurity Lab
permalink: /projects/Enterprise-Active-Directory-lab/
---
# Enterprise Active Directory Cybersecurity Lab

lab focused on Windows Active Directory administration, security monitoring, attack simulation, and detection engineering fundamentals.

## Environment used:
* Windows Server 2025 Domain Controller
* Windows 10 as domain-joined workstation
* same as always Kali Linux as attacker machine
* Virtualbox with internal network

First thing to do after installation both machines and Active Directory on server was to promote it to domain controller

![promote](SecurityProjects/images/active-directory-lab/promote to domain controller.png)
named it corp.local

then I created new Organizational Units, groups and users in them

![ou](SecurityProjects/images/active-directory-lab/nowe OU.png)
![grupy](SecurityProjects/images/active-directory-lab/nowe grupy.png)
![usery](SecurityProjects/images/active-directory-lab/nowe usery.png)

next it was time to edit Group and Lockout Policy 

![hasła](SecurityProjects/images/active-directory-lab/gpo  hasła.png)
![lockout](SecurityProjects/images/active-directory-lab/lockout policy.png)


