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

![promote](/SecurityProjects/images/promote-to-domain-controller.png)

named it corp.local

then I created new Organizational Units, groups and users in them

![ou](/SecurityProjects/images/active-directory-lab/nowe-OU.png)
![grupy](/SecurityProjects/images/active-directory-lab/nowe-grupy.png)
![usery](/SecurityProjects/images/active-directory-lab/nowe-usery.png)

next it was time to edit Group and Lockout Policy 

![hasła](/SecurityProjects/images/active-directory-lab/gpo-hasła.png)
![lockout](/SecurityProjects/images/active-directory-lab/lockout-policy.png)

next I activated audit `logon` and `powershell logs` and forced Group Policy update by 
```Powershell
gpupdate /force
```

![hasła](/SecurityProjects/images/active-directory-lab/logi-z-logowania.png)
![hasła](/SecurityProjects/images/active-directory-lab/powershell-logi.png)





As workstation and domain controller was all set I did some basic attack simulation with monitoring in `Event Viewer`
For the attack I installed `Kerbrute` on Kali

`Kerbrute` is a great tool for that as it bruteforce valid AD accounts through Kerberos Pre-Authentication. 

`Kerberos` is an authentication protocol designed to verify user or host identities over insecure networks; **it's also native protocol in Active Directory**

Firstly, I used `userenum` command with basic users list(few correct and few incorrect) which only enumerates valid usernames, doesn't try to log-in;

```Bash
./kerbrute userenum --dc 192.168.56.10 -d corp.local users.txt
```

![hasła](/SecurityProjects/images/active-directory-lab/enum1.png)

so in Event Viewer the only event generated should be `4768` - Kerberos Authentication Ticket (TGT) requested
But there was none. I needed to dig a little to found that my Event Viewer didn't raport any events as Windows auditing was not enabled for Kerberos authentication events on the Domain Controller.


![hasła](/SecurityProjects/images/active-directory-lab/brak-audytu.png)

fast correction to that:

![hasła](/SecurityProjects/images/active-directory-lab/kerberos-kategorie-ustawione.png)

I checked that by some password spraying:

```Bash
./kerbrute passwordspray --dc 192.168.56.10 -d corp.local users.txt 'Password123!'
```

![hasła](/SecurityProjects/images/active-directory-lab/spray.png)

which worked correctly:

`4771` - kerberos pre-authentication failed

![hasła](/SecurityProjects/images/active-directory-lab/audit-failure.png)

now something to boost our logging system - `Sysmon` installation

`Sysmon` is a  Windows system service and device driver  that provides detailed, high-granularity logging of system activity

Installation process is simply one exe file with one markup so I wont bother you with that

Afterwards it is time for some tests:

`Get-Process` in powershell to list processes and trigger activity - worked

![hasła](/SecurityProjects/images/active-directory-lab/simple-sysmon-test.png)


now something more sophisticated:

```Bash
-nop -w hidden -c "whoami"
```

`-nop` - no profile

`-w hidden` - hidden window

`-c` - execute command at this case "whoami"

![hasła](/SecurityProjects/images/active-directory-lab/whoami-w-event-viewer.png)

 sysmon catched that perfectly



