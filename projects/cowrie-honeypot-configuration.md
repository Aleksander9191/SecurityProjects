# Cowrie SSH & Telnet Honeypot Deployment

## Introduction

In this project, I deployed **Cowrie**, a medium-interaction SSH and Telnet honeypot, on a Debian 13 VPS.

The objective is to expose simulated remote access services to the internet, collect authentication attempts and attacker activity, and analyze the telemetry generated during interactive sessions.

The deployment focuses on secure service isolation, controlled network exposure, and reliable event collection.

---

## Lab Architecture

|Layer|Component|
|---|---|
|**Operating System**|Debian 13 (Minimal)|
|**Honeypot**|Cowrie|
|**Python Environment**|Python virtual environment|
|**Service Management**|systemd|
|**Network Filtering**|UFW|
|**Outbound Protection**|nftables|
|**Log Format**|JSON|

### Exposed Services

|Service|Port|Purpose|
|---|---|---|
|Administrative SSH|TCP 22|Secure VPS administration|
|Cowrie SSH|TCP 2222|SSH honeypot|
|Cowrie Telnet|TCP 23|Telnet honeypot|

---

## Cowrie Installation

Cowrie was installed in a dedicated directory:

```
/opt/cowrie
```

A separate Python virtual environment was created to isolate Cowrie and its dependencies from the system Python installation.

The honeypot runs under a dedicated, unprivileged Linux account:

```
User: cowrie
Group: cowrie
```

This separation prevents the honeypot service from running with root privileges.

---

## Service Configuration

Cowrie was configured as a systemd service to provide automatic startup and centralized service management.

The service runs under the dedicated `cowrie` account.

The service status can be verified using:

```
sudo systemctl status cowrie
```

The service was successfully started and verified as active.

---

## SSH & Telnet Configuration

### SSH Honeypot

The SSH service listens on TCP port `2222`.

This allows the honeypot to expose an SSH endpoint without interfering with the administrative SSH service running on port `22`.

### Telnet Honeypot

Telnet support was enabled in the Cowrie configuration.

The listener was configured to use the standard Telnet port:

```
[telnet]
enabled = true
listen_endpoints = tcp:23:interface=0.0.0.0
```

### Binding to a Privileged Port

On Linux, ports below `1024` require elevated privileges or the `CAP_NET_BIND_SERVICE` capability.

Instead of running Cowrie as root, the required capability was assigned to its systemd service:

```
[Service]
AmbientCapabilities=CAP_NET_BIND_SERVICE
```

This allows the service to bind to TCP port `23` while retaining its unprivileged execution context.

---

## Network Security Configuration

### UFW Firewall

UFW was configured with a default-deny incoming policy and an allow-outgoing policy.

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Only the required TCP ports were allowed:

```
sudo ufw allow 22/tcp
sudo ufw allow 23/tcp
sudo ufw allow 2222/tcp
```
![screen](../images/cowrie-config/ufw.png)

The rules apply to both IPv4 and IPv6.

### Outbound Traffic Protection

An additional nftables rule was implemented to restrict outbound connections initiated by the Cowrie service account.

The rule blocks new outbound connections associated with the Cowrie UID while allowing responses belonging to established incoming connections.

This provides an additional layer of containment against potential abuse of the simulated shell.

---

## Service Validation

The active listening sockets were verified using:

```
sudo ss -lntp
```

The expected listeners were confirmed:

```
0.0.0.0:23
0.0.0.0:2222
```

External connectivity was tested from a Windows workstation using PowerShell:

```
Test-NetConnection <VPS_IP> -Port 23
Test-NetConnection <VPS_IP> -Port 2222
```

The Telnet connectivity test returned:

```
TcpTestSucceeded : True
```

This confirmed that the Telnet service was reachable externally.

---

## Event Logging

Cowrie stores generated events in JSON format.

The primary log file is located at:

```
/opt/cowrie/var/log/cowrie/cowrie.json
```

The log contains structured information about honeypot activity, including:

- Incoming connections and session metadata.
    
- Authentication attempts and successful logins.
    
- Commands entered during interactive sessions.
    
- Session termination events.
    

### Live Log Monitoring

The log can be monitored in real time using:

```
sudo tail -f /opt/cowrie/var/log/cowrie/cowrie.json
```

For structured event inspection, `jq` can be used:

```
sudo jq -c 'select(.eventid=="cowrie.login.success")' \
/opt/cowrie/var/log/cowrie/cowrie.json
```

---

## Telnet Session Validation

A test connection was established using PuTTY in Telnet mode.

A simulated login session was successfully initiated, and several commands were entered:

```
pwd
uname -a
whoami
exit
```
![screen](../images/cowrie-config/telnet.png)

Cowrie recorded the corresponding authentication, command input, and session termination events.

The generated events included:

|Event ID|Description|
|---|---|
|`cowrie.session.connect`|New session established|
|`cowrie.login.success`|Authentication accepted by the honeypot|
|`cowrie.command.input`|Command entered during the session|
|`cowrie.session.closed`|Session terminated|

The session was successfully captured in the JSON event log.

---

## Summary

In this initial phase, I deployed Cowrie on a Debian 13 VPS and configured both SSH and Telnet honeypot services.

The deployment includes:

- Dedicated unprivileged service account.
    
- Isolated Python environment.
    
- systemd service management.
    
- SSH and Telnet listeners.
    
- UFW firewall configuration.
    
- nftables outbound connection restrictions.
    
- JSON-based event collection.
    
- External connectivity and session validation.
    

The next phase will focus on extending the honeypot infrastructure with additional exposed services and developing custom tools for telemetry analysis, threat investigation, and visualization.

---

## Future Development

Planned extensions:

- HTTP/HTTPS honeypot.
    
- FTP honeypot.
    
- ICS/OT honeypot (Modbus TCP).
    
- Python-based Cowrie log analyzer.
    
- Attack source GeoIP and ASN enrichment.
    
- Threat Intelligence integration.
    
- Custom detection rules using Sigma.
    
- YARA-based file analysis.
    
- Centralized log management and dashboards.
