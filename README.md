# Home SOC Lab

This repository documents my home SOC lab built around Splunk Cloud. The goal of this project is to better understand how endpoint telemetry is collected, forwarded, and analyzed in a SIEM while building practical detection engineering skills.

The lab currently consists of a Windows 11 endpoint forwarding Windows Event Logs and Sysmon telemetry to Splunk Cloud using the Splunk Universal Forwarder. As the project grows, I plan to add custom detections, dashboards, attack simulations, and Active Directory.

## Current Lab

- Windows 11
- Splunk Cloud
- Splunk Universal Forwarder
- Windows Event Logs
  - Security
  - System
  - Application
- Sysmon

## Architecture

> *(Insert architecture diagram here)*

## Current Progress

- [x] Configure Splunk Cloud
- [x] Install Splunk Universal Forwarder
- [x] Forward Windows Event Logs
- [x] Install and configure Sysmon
- [x] Verify telemetry ingestion
- [ ] Create detection rules
- [ ] Build dashboards
- [ ] Simulate attacks
- [ ] Map detections to MITRE ATT&CK

## Screenshots

### Windows Event Logs

*(Insert screenshot)*

### Sysmon Events

*(Insert screenshot)*

### Splunk Searches

*(Insert screenshot)*

## Repository Structure

```
architecture/
docs/
screenshots/
spl/
detections/
```

## Roadmap

### Phase 1
- Windows Event Log collection
- Sysmon deployment
- Splunk Cloud integration

### Phase 2
- Failed logon detection
- PowerShell detection
- Registry monitoring

### Phase 3
- Atomic Red Team
- Dashboards
- MITRE ATT&CK mapping

### Phase 4
- Active Directory
- Multi-endpoint monitoring
- Threat hunting scenarios
