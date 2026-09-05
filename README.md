# Cybersecurity Portfolio


![Security+](https://img.shields.io/badge/CompTIA-Security%2B-red)
![Elastic Security](https://img.shields.io/badge/Elastic-Security-005571)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-blue)
![Sysmon](https://img.shields.io/badge/Sysmon-Enabled-success)
![Zabbix](https://img.shields.io/badge/Zabbix-Monitoring-CC0000?logo=zabbix&logoColor=white)

A collection of hands-on cybersecurity, detection engineering and system administration projects built to demonstrate practical blue team skills.

The portfolio focuses on realistic lab environments, attack simulation, detection engineering, Windows internals, Active Directory and SIEM technologies.

---

## Portfolio Website

Visit the portfolio:

➡️ https://aleksander9191.github.io/SecurityProjects/

---

# Featured Projects

## Linux Server Hardening

Linux administration project focused on server hardening and SSH attack detection.

Topics:

- SSH Hardening
- Fail2ban
- UFW Firewall
- Log Analysis
- Brute Force Detection

---

## Enterprise Active Directory Lab

Enterprise-style Active Directory environment built from scratch.

Highlights:

- Windows Server 2022
- Active Directory
- DNS
- Group Policy
- Windows 10 Clients
- Enterprise network simulation

---

## Zabbix Monitoring Lab

Hands-on infrastructure monitoring lab focused on Windows and Linux system monitoring, alerting and troubleshooting.

Highlights:

- Zabbix Server
- Windows monitoring
- CPU, memory and disk monitoring
- Service monitoring
- Windows Event Log monitoring
- Custom triggers
- Network interface monitoring
- Dashboard creation
- Troubleshooting automatic updates and stuck systemd jobs

The lab also included an unplanned troubleshooting scenario where an automatic system update became stuck during service restarts, causing MySQL and Zabbix Server to become unavailable. The issue was investigated through `systemctl`, `journalctl`, process trees and systemd job queues before restoring the affected services.

---

## Linux Security Monitoring with auditd & Elastic

Detection engineering project focused on Linux security monitoring using the native Linux Audit Framework (`auditd`) integrated with the Elastic Stack.

Highlights:

- Linux Audit Framework (`auditd`)
- Elastic Agent & Fleet
- Elasticsearch
- Kibana
- Detection Engineering
- Linux Security Monitoring
- Audit Rule Development
- Event Validation & Investigation

The project demonstrates how security-relevant Linux events are generated, collected, forwarded to Elastic, and analyzed from a SOC analyst perspective. It includes practical audit rule development, local event validation with `ausearch`, and detailed investigation of audit telemetry within Kibana.

---

## Elastic Detection Lab

Detection Engineering project using:

- Elastic Security
- Elasticsearch
- Kibana
- Sysmon
- Atomic Red Team

Implemented custom detections for MITRE ATT&CK techniques including:

| Technique | Status |
|-----------|:------:|
| [T1087.001 – Local Account Discovery](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1087.001-Local-Account-Discovery/README.md) | ✅ |
| [T1059.001 – PowerShell Encoded Command Execution](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1059.001-PowerShell/README.md) | ✅ |
| [T1110.001 – Password Guessing](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1110.001%20-%20Password%20Guessing/README.md) | ✅ |
| [T1218.010 – Regsvr32 Scriptlet Execution](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1218.010%20-%20Regsvr32%20Scriptlet%20Execution/README.md) | ✅ |
| [T1105 – Ingress Tool Transfer](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1105%20-%20Ingress%20Tool%20Transfer/README.md) | ✅ |
| [T1685.005 – Clear Windows Event Logs](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1685.005%20-%20Disable%20or%20Modify%20Tools%3A%20Clear%20Windows%20Event%20Logs/README.md) | ✅ |
| [T1003.001 – LSASS Memory Dump](./T1003.001-LSASS-Memory-Dump/README.md) | ✅ |
| [T1027 – PowerShell Character Array Obfuscation](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1027%20-%20PowerShell%20Character%20Array%20Obfuscation/README.md) | ✅ |
| [T1564.001 – Hidden Files and Directories](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1564.001%20%E2%80%93%20Hide%20Artifacts%3A%20Hidden%20Files%20and%20Directories/README.md) | ✅ |
| [T1053.005 – Scheduled Task](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1053.005%20-%20Scheduled%20Task/README.md) | ✅ |
| [T1547.001 – Winlogon and Registry Run Key Persistence Detection](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1547.001%20-%20Winlogon%20and%20Registry%20Run%20Key%20Persistence%20Detection/README.md) | ✅ |
| [T1543.003 – Windows Service Persistence Detection](https://github.com/Aleksander9191/Elastic-Detection-Lab/blob/main/techniques/T1543.003%20Windows%20Service%20Persistence%20Detection/README.md) | ✅ |

Each detection includes:

- Attack simulation
- Sysmon investigation
- Custom Elastic rule
- Alert validation
- Documentation

---

# Technologies

### SIEM

- Elastic Security
- Elasticsearch
- Kibana

### Endpoint Telemetry

- Sysmon
- Elastic Agent

### Windows

- Active Directory
- Group Policy
- Windows Event Logs
- PowerShell

### Linux

- Ubuntu Server
- SSH
- UFW
- Fail2ban

### Offensive Simulation

- Atomic Red Team
- MITRE ATT&CK

---



# Goal

The purpose of this repository is to document my journey toward becoming a SOC Analyst  through practical, reproducible projects based on real-world attack techniques.
