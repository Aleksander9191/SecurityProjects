
#  Monitoring Lab with Zabbix

## Overview

This project demonstrates a small Windows monitoring environment built with **Zabbix**.

The goal of the lab was to monitor a Windows 10 workstation, collect system performance metrics and Windows Event Logs, and create custom triggers capable of detecting both operational and security-related events.


## Lab Architecture

| Component | Value |
|-----------|-------|
| **Monitored Host** | Windows 10 workstation (`Win10_1`) |
| **Monitoring Platform** | Zabbix 7.4.14 |
| **Monitored Components** | CPU, Memory, Disk, Services, Network Interfaces, Windows Security Event Log |


The project was intentionally kept small and focused on demonstrating the core capabilities of Zabbix rather than building a complete enterprise monitoring environment.
After setting up zabbix on a server and installing agents on workstations I proceeded to test Zabbix features

![1](../images/start.png)

---


## Service Monitoring

Windows services are monitored using Zabbix service discovery.

The lab includes testing service availability by stopping and starting services.

### Example: Print Spooler

The `Spooler` service was stopped manually to verify whether Zabbix could detect the failure.

The following trigger was used:

```
Windows: "Spooler" (Print Spooler) is not running
```


After the service remained stopped for the required number of checks, Zabbix generated a problem event.

![1](../images/spooler-problem.png)

The service was then started again and the problem automatically recovered.

This demonstrated the complete monitoring cycle:

```
Service running
      ↓
Service stopped
      ↓
Zabbix detects failure
      ↓
PROBLEM
      ↓
Service started
      ↓
OK / Recovery
```

---

## Disk Monitoring

Disk activity was tested by intentionally generating a large amount of disk I/O on the Windows workstation.

A PowerShell background job was used to repeatedly write data to the disk.

Example:

```powershell
$job = Start-Job {
    $end = (Get-Date).AddMinutes(2)
    $path = "$env:TEMP\disk-test.bin"
    $buffer = New-Object byte[] (10MB)

    while ((Get-Date) -lt $end) {
        [System.IO.File]::WriteAllBytes($path, $buffer)
    }

    Remove-Item $path -Force
}
```

The generated disk load caused disk utilization to increase significantly, reaching approximately 100% during the test.

Zabbix detected the condition and generated:

```
Windows: 0 C: Disk is overloaded
```

![1](../images/diskc-full.png)

This demonstrated that Zabbix can detect abnormal resource utilization caused by an actual workload rather than simply displaying historical data.

---

## Memory Monitoring

Memory utilization was tested by generating a sustained memory load on the Windows workstation.

The Zabbix dashboard showed the resulting increase in memory utilization.

A trigger was configured to detect sustained high memory usage:

```
avg(/Win10_1/vm.memory.util,1m)>50
```



![1](../images/memory-trigger.png)


the test was performed using the command

```powershell
1..4 | ForEach-Object { Start-Job { while ($true) { [Math]::Sqrt(123456789) } } }
```

It creates four background jobs, each of which performs an infinite loop to calculate the square root of 123456789. 
This is a simple way to generate constant CPU load for monitoring tests.

![1](../images/4jobs.png)

The test demonstrated how Zabbix can distinguish between a short-lived spike and sustained resource exhaustion.

---

# Windows Security Event Monitoring

One of the main objectives of the lab was to use Zabbix to monitor Windows Security Event Logs.

Instead of collecting the entire Security log, dedicated Zabbix items were created for specific Event IDs.

This allows Zabbix to act not only as a performance monitoring system, but also as a basic event detection mechanism.

---

## Failed Logon Detection — Event ID 4625

A Zabbix item was created to collect failed Windows logon events.

![1](../images/failed-logon-item.png)

### Item

```
Security log: Failed logon events
```

The item uses the following Zabbix agent active key:

```
eventlog[Security,,,,4625,,skip]
```

The key can be understood as:

```
eventlog[
    Security,   ← Windows Security log
    ,           ← any event text
    ,           ← any severity
    ,           ← any source
    4625,       ← Event ID 4625
    ,           ← default limit
    skip        ← skip old events
]
```

The `skip` parameter is important because it prevents Zabbix from processing old events already present in the Security log when the item starts monitoring.

### Test

Failed authentication attempts were generated intentionally.

Windows generated Event ID:

```
4625
```

which represents a failed logon attempt.

![1](../images/failed-logon-event.png)

Zabbix successfully collected the event and displayed the complete event information, including:

- account name
- domain
- logon type
- failure reason
- source address
- process information

A trigger was then created to detect the event.

### Result

Zabbix generated:

```
Multiple failed logon attempts detected
```

![1](../images/failed-logon-problem.png)

---

# User Account Creation Detection — Event ID 4720

The second Windows security event monitored in the lab was user account creation.

### Item

```
Security log: User account created
```

The item collects:

```
eventlog[Security,,,,4720,,skip]
```

Event ID `4720` is generated when a new Windows user account is created.

### Test

A test account was created using PowerShell:

```cmd
net user ZabbixTest "ZabbixTest!2026asdasdasd" /add
```

Windows generated Event ID:

```
4720
```

The event was successfully collected by Zabbix.

A trigger was configured using:

```
logeventid(/Win10_1/eventlog[Security,,,,4720,,skip])=4720
```

### Result

Zabbix generated:

```
Security: User account created
```

![1](../images/user-account-created.png)

This demonstrated how specific Windows security events can be transformed into actionable Zabbix problems.

---

# Dashboard

A exemplary Zabbix dashboard was created for the Windows 10 monitoring lab.

The dashboard provides a single overview of the monitored workstation.

It contains:

- CPU utilization graph
- Memory utilization graph
- Disk utilization graph
- Current problems
- Zabbix agent availability



### Dashboard Preview

![1](../images/dashboard.png)

---

## Detection Scenarios

| Scenario | Windows/Zabbix event | Result |
|----------|----------------------|--------|
| High CPU utilization | Performance metric | Alert generated |
| High memory utilization | Performance metric | Alert generated |
| Heavy disk activity | Disk utilization | Alert generated |
| Print Spooler stopped | Windows service state | Alert generated |
| Failed logon | Event ID 4625 | Security alert generated |
| User account creation | Event ID 4720 | Security alert generated |
| Zabbix Agent unavailable | Agent availability | Alert generated |

---

## What This Lab Demonstrates

This project demonstrates several practical Zabbix capabilities:

- Windows host monitoring
- Zabbix Agent monitoring
- Active Zabbix Agent items
- Automatic Windows service discovery
- Performance monitoring
- Resource exhaustion detection
- Windows Event Log collection
- Event ID filtering
- Custom triggers
- Problem and recovery states
- Dashboard creation
- Basic security event monitoring

An important part of the project was understanding the difference between **collecting data** and **detecting a condition**.

For example:

```
Item
 ↓
Collects data

Trigger
 ↓
Evaluates collected data

Problem
 ↓
Represents a detected condition
```

This separation is fundamental to how Zabbix monitoring works.

---

## Lessons Learned

During the lab,  two  practical issues were encountered.

### Windows Event Log Auditing

Not every Windows security event is necessarily generated by default.

For example, Event ID `4720` required the appropriate Windows auditing policy to be enabled.

The category was:

```
Account Management
```

with the subcategory:

```
User Account Management
```

After enabling the required auditing, Event ID 4720 was generated and successfully collected by Zabbix.

## Troubleshooting: Zabbix Unavailable After Unattended Upgrades

During the lab, the Zabbix frontend unexpectedly became unavailable. Initial investigation suggested that MySQL or Zabbix Server might have failed, but service logs showed that MySQL had been shut down cleanly rather than crashing.

---

### Investigation

The first step was checking the status of the main Zabbix components:

```bash
sudo systemctl status mysql --no-pager
sudo systemctl status zabbix-server --no-pager
sudo systemctl status apache2 --no-pager
```

Apache was running, while MySQL was inactive and Zabbix Server was stuck in a `deactivating` state.

The MySQL error log showed:

```text
Received SHUTDOWN from user. Shutting down mysqld
Shutdown complete
```

This confirmed that MySQL had received a normal shutdown request and that there was no evidence of a crash or InnoDB corruption.

The next step was checking the systemd job queue:

```bash
sudo systemctl list-jobs
```

This revealed:

```text
mysql.service             start     waiting
zabbix-server.service     restart   running
apt-daily-upgrade.service start     running
```

At the same time, `apt-daily-upgrade` had been running for approximately 18 minutes without producing new log entries.

Further investigation showed that the automatic update process was executing:

```text
unattended-upgrade
    └── needrestart
        └── systemctl restart apache2.service mysql.service ...
```

The `needrestart` process had attempted to restart multiple services, including MySQL and Zabbix Server, but the restart operation had become stuck.

The package manager itself was then checked:

```bash
sudo dpkg --audit
```

The command returned no output, confirming that the package database was in a consistent state.

The update log also showed that packages such as `libssl3t64` and `openssl` had been successfully installed before the process reached the service restart stage.

---

### Root Cause

The issue was **not** caused by Zabbix or MySQL itself.

The actual chain of events was:

```text
unattended-upgrade
        ↓
package updates completed
        ↓
needrestart
        ↓
restart multiple services
        ↓
systemctl restart mysql + zabbix-server + other services
        ↓
restart operation became stuck
        ↓
MySQL job → waiting
        ↓
Zabbix Server job → running/waiting
        ↓
Zabbix frontend became unavailable
```

This was confirmed by the `apt-daily-upgrade` logs, which showed the update process reaching the **Restarting services...** phase and invoking `systemctl restart` for MySQL, Zabbix Server and other services.

---

### Resolution

Because the package installation had already completed successfully and `dpkg --audit` reported no issues, the stuck update service was stopped:

```bash
sudo systemctl stop apt-daily-upgrade.service
```

The remaining stale systemd jobs were then identified:

```bash
sudo systemctl list-jobs
```

They were manually cancelled:

```bash
sudo systemctl cancel 2656
sudo systemctl cancel 3460
```

After this, systemd reported:

```text
No jobs running.
```

MySQL could then be started normally:

```bash
sudo systemctl start mysql
sudo systemctl status mysql --no-pager
```

The service returned to:

```text
Active: active (running)
Status: "Server is operational"
```

The Zabbix Server was subsequently started again:



---

### Lessons Learned

This incident provided an unplanned but valuable troubleshooting scenario.

The most important lesson was that an application appearing to be unavailable does **not necessarily mean that the application itself has failed**. The investigation had to follow the complete dependency chain:

```text
Zabbix frontend unavailable
        ↓
Apache
        ↓
MySQL
        ↓
systemd
        ↓
apt-daily-upgrade
        ↓
unattended-upgrade
        ↓
needrestart
        ↓
stuck service restart
```

The incident demonstrated a practical troubleshooting workflow based on:

- identifying the visible symptom,
- checking service status,
- examining system logs with `journalctl`,
- inspecting the systemd job queue,
- tracing parent/child processes,
- verifying package consistency with `dpkg --audit`,
- identifying the actual root cause,
- clearing stale jobs,
- restoring services in the correct dependency order.

This was particularly valuable because the final cause was **external to Zabbix itself**: the monitoring server became unavailable as a side effect of a stuck automatic system update and service restart process.
## Conclusion

This lab started as a basic Windows resource monitoring setup and was extended with service monitoring and Windows Security Event Log detection.

The final environment demonstrates that Zabbix can be used for more than traditional infrastructure monitoring.

It can also provide a lightweight form of security event detection by:

1. collecting selected Windows Event Log events,
2. filtering them by Event ID,
3. evaluating them with triggers,
4. presenting the result as actionable problems.

The project intentionally remains small and focused, providing a foundation for a larger Windows security monitoring environment without requiring a large number of virtual machines.
