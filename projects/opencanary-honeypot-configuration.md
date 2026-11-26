# OpenCanary HTTP & HTTPS Honeypot Deployment

## Introduction

This deployment is a **continuation of the existing Cowrie Honeypot project**.

The objective is to expand the already deployed honeypot infrastructure by introducing additional exposed services for monitoring and analyzing internet activity.

Instead of deploying another VPS or replacing the existing Cowrie installation, **OpenCanary was deployed as a second, independent honeypot on the same Debian 13 machine**.

Cowrie continues to provide SSH and Telnet emulation, while OpenCanary introduces HTTP and HTTPS services.

Both honeypots operate independently, using separate Linux accounts, Python virtual environments, configurations, and systemd services.

The deployment focuses on service isolation, controlled network exposure, and reliable collection of structured JSON events.

---

## Lab Architecture

|Layer|Component|
|---|---|
|**Operating System**|Debian 13 (Minimal)|
|**Existing Honeypot**|Cowrie – SSH and Telnet|
|**Additional Honeypot**|OpenCanary – HTTP and HTTPS|
|**Python Environment**|Separate virtual environments|
|**Service Management**|systemd|
|**Network Filtering**|UFW|
|**Outbound Protection**|nftables (Cowrie)|
|**Log Format**|JSON|

### Exposed Services

The existing Cowrie deployment remains unchanged. OpenCanary introduces two additional network services:

|Service|Port|Purpose|
|---|---|---|
|Administrative SSH|TCP 22|Secure VPS administration|
|Cowrie SSH|TCP 2222|SSH honeypot|
|Cowrie Telnet|TCP 23|Telnet honeypot|
|OpenCanary HTTP|TCP 80|HTTP honeypot|
|OpenCanary HTTPS|TCP 443|HTTPS honeypot|

### Service Separation

OpenCanary was installed in a dedicated directory:

```
/opt/opencanary
```

A separate Python virtual environment was created to isolate its dependencies from both Cowrie and the system Python installation.

The service runs under its own unprivileged Linux account:

```
User: opencanary
Group: opencanary
```

This allows both honeypots to operate independently on the same host.

---

## OpenCanary Installation

OpenCanary was installed using a dedicated Python virtual environment.

The installed environment uses:

|Component|Version|
|---|---|
|Python|3.13.5|
|OpenCanary|0.9.10|
|Twisted|26.4.0|

The package was installed using:

```
sudo -u opencanary /opt/opencanary/venv/bin/python -m pip install opencanary
```

The default configuration was copied to:

```
/etc/opencanaryd/opencanary.conf
```

The configuration was then adjusted to enable HTTP and HTTPS modules.

---

## HTTP & HTTPS Configuration

OpenCanary was configured to expose two web services:

### HTTP Honeypot

The HTTP module listens on TCP port `80`.

### HTTPS Honeypot

The HTTPS module listens on TCP port `443`.

Both services use the `nasLogin` skin to emulate a NAS login interface.

The HTTP server banner was configured as:

```
Apache/2.2.22 (Ubuntu)
```

This is an emulated server identification string and does not represent an actual Apache installation.

### Module Configuration

Relevant configuration options in `opencanary.conf`:

```
"http.enabled": true,
"http.port": 80,
"http.skin": "nasLogin",

"https.enabled": true,
"https.port": 443,
"https.skin": "nasLogin",

"ftp.enabled": false
```

FTP and other unused modules remain disabled.

---

## HTTPS Certificate Configuration

A self-signed TLS certificate was generated for the HTTPS service.

Certificate files are stored in:

```
/etc/ssl/opencanary/
```

The certificate and private key are:

```
opencanary.pem
opencanary.key
```

The certificate was generated using OpenSSL:

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/opencanary/opencanary.key \
  -out /etc/ssl/opencanary/opencanary.pem \
  -subj "/CN=localhost"
```

The private key is owned by `root:opencanary` and has restricted read permissions.

The certificate is self-signed. Clients connecting through a browser will therefore receive a certificate trust warning.

---

## Service Configuration

OpenCanary was configured as a separate systemd service:

```
/etc/systemd/system/opencanary.service
```

The service runs under the dedicated `opencanary` account.

### Binding to Privileged Ports

On Linux, ports below `1024` require elevated privileges or the `CAP_NET_BIND_SERVICE` capability.

Instead of running OpenCanary as root, the required capability was assigned through systemd:

```
[Service]
User=opencanary
Group=opencanary

AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
```

This allows OpenCanary to bind to TCP ports `80` and `443` while retaining an unprivileged execution context.

The service is managed independently from Cowrie.

### Service Management

Reload systemd configuration:

```
sudo systemctl daemon-reload
```

Start OpenCanary:

```
sudo systemctl start opencanary
```

Enable automatic startup:

```
sudo systemctl enable opencanary
```

Check service status:

```
sudo systemctl status opencanary
```

---

## Network Security Configuration

### UFW Firewall

The existing UFW configuration was extended to allow incoming HTTP and HTTPS connections.

```
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

The firewall retains the existing default-deny incoming policy.

The rules apply to both IPv4 and IPv6.

### Service Isolation

OpenCanary operates under a separate Linux account and Python environment.

This provides separation from the existing Cowrie service and prevents OpenCanary from requiring root privileges to operate.

---

## Service Validation

The active listening sockets were verified using:

```
sudo ss -tulpn | grep -E ':80|:443'
```

The expected listeners were confirmed:

```
0.0.0.0:80
0.0.0.0:443
```

### Local Connectivity Tests

HTTP:

```
curl -I http://127.0.0.1
```

HTTPS:

```
curl -k -I https://127.0.0.1
```

The `-k` option was used to bypass certificate verification because the HTTPS service uses a self-signed certificate.

### External Connectivity Tests

External connectivity was tested from a Windows workstation:

```
curl.exe -i http://<VPS_IP>/index.html
curl.exe -k -i https://<VPS_IP>/index.html
```

Both services returned HTTP responses and were confirmed reachable over the public internet.

---

## Event Logging

OpenCanary stores generated events in JSON format.

The primary log file is located at:

```
/var/log/opencanary/opencanary.log
```

The file logger was configured in `opencanary.conf`.

### Observed Event Types

|Log Type|Description|
|---|---|
|`1001`|OpenCanary service/status event|
|`3000`|HTTP GET request|
|`3001`|HTTP POST login attempt|

### Recorded Event Information

HTTP and HTTPS events may contain:

- Source IP address and source port.
    
- Destination host and destination port.
    
- Requested URL path.
    
- HTTP User-Agent.
    
- Login username and submitted password (POST events).
    
- Event type and timestamp.
    
- Configured honeypot skin.
    

### Live Log Monitoring

The log can be monitored in real time using:

```
sudo tail -f /var/log/opencanary/opencanary.log
```

Service logs can also be inspected through systemd:

```
sudo journalctl -u opencanary -f
```

### HTTP GET Validation

A test HTTP GET request generated an event with `logtype: 3000`.

### HTTPS GET Validation

A test HTTPS GET request generated an event with `logtype: 3000` and destination port `443`.

### HTTPS Login Attempt Validation

A test login attempt was submitted using dummy credentials:

```
curl -k -i -X POST \
  -d "username=honeypot_test&password=FakePassword123!" \
  https://127.0.0.1/index.html
```

OpenCanary successfully recorded the attempt as `logtype: 3001`.

**Security note:** POST events may contain submitted credentials in plaintext. Log files must be protected, and only dummy credentials should be used during testing.

---

## Initial HTTP/HTTPS Exposure

Both OpenCanary services were tested from an external Windows workstation.

The generated events confirmed that the VPS was reachable over the public internet.

The following activity was observed:

|Protocol|Destination Port|Event Type|Result|
|---|---|---|---|
|HTTP|80|`3000` – GET|Successfully recorded|
|HTTPS|443|`3000` – GET|Successfully recorded|
|HTTPS|443|`3001` – POST login attempt|Successfully recorded|

The source IP address recorded by OpenCanary matched the external client address used during testing.

The tests confirmed that both services were operational and that incoming requests were stored in the JSON event log.

---

## Integration with the Existing Honeypot Project

OpenCanary extends the existing Cowrie deployment without replacing or modifying its SSH and Telnet services.

The current infrastructure now provides four exposed honeypot services:

- SSH – TCP 2222 (Cowrie).
    
- Telnet – TCP 23 (Cowrie).
    
- HTTP – TCP 80 (OpenCanary).
    
- HTTPS – TCP 443 (OpenCanary).
    

The next development step is to integrate OpenCanary telemetry into the existing Python-based analysis workflow.

The planned analysis will include source IP enrichment, ASN and country statistics, HTTP request paths, User-Agent analysis, and login attempt statistics.

---

## Future Development

Planned extensions:

- FTP honeypot.
    
- Python-based OpenCanary log analyzer.
    
- Integration of Cowrie and OpenCanary telemetry.
    
- Comparative analysis of exposed services.
    
- Threat Intelligence integration.
    
- Custom detection rules using Sigma.
    
- YARA-based file analysis.
    
- Centralized log management and dashboards.
    
- ICS/OT honeypot (Modbus TCP).
