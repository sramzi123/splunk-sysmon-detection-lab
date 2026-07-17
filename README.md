# Home SOC Lab

<<<<<<< HEAD
This repository documents my home SOC lab, built to better understand how endpoint telemetry is collected, forwarded, and analyzed in a SIEM, while building practical detection engineering skills.

The lab currently consists of a Windows 11 endpoint forwarding Windows Event Logs and Sysmon telemetry through a Splunk Universal Forwarder into a local Splunk Enterprise instance. As the project grows, I plan to add custom detections, dashboards, attack simulations, and Active Directory.
=======
This repository documents my home SOC lab built around Splunk Cloud. The goal of this project is to better understand how endpoint telemetry is collected, forwarded, and analyzed in a SIEM while building practical detection engineering skills.

The lab currently consists of a Windows 11 endpoint forwarding Windows Event Logs and Sysmon telemetry to Splunk Cloud using the Splunk Universal Forwarder. As the project grows, I plan to add custom detections, dashboards, attack simulations, and Active Directory.
>>>>>>> ced275fa66f7c8a99a180a8e364c413ac56f09e9

## Current Lab

- Windows 11
<<<<<<< HEAD
- Splunk Enterprise (local instance)
=======
- Splunk Cloud
>>>>>>> ced275fa66f7c8a99a180a8e364c413ac56f09e9
- Splunk Universal Forwarder
- Windows Event Logs
  - Security
  - System
  - Application
- Sysmon

## Architecture

<<<<<<< HEAD
![Architecture diagram](architecture/homelab-architecture.png)

## Documentation

- [Setup Guide](docs/setup.md) — Sysmon, Universal Forwarder, and TA configuration
- [Troubleshooting: Sourcetype Conflict](docs/troubleshooting.md) — diagnosing a real config conflict between two overlapping Sysmon add-ons
- [Investigation: 4625 Baseline vs Anomaly](docs/analysis-4625-baseline-vs-anomaly.md) — reading failed logon events carefully instead of taking them at face value

## Current Progress

- [x] Install and configure Splunk Enterprise
=======
> *(Insert architecture diagram here)*

## Current Progress

- [x] Configure Splunk Cloud
>>>>>>> ced275fa66f7c8a99a180a8e364c413ac56f09e9
- [x] Install Splunk Universal Forwarder
- [x] Forward Windows Event Logs
- [x] Install and configure Sysmon
- [x] Verify telemetry ingestion
<<<<<<< HEAD
- [x] Diagnose and fix a sourcetype parsing conflict
=======
>>>>>>> ced275fa66f7c8a99a180a8e364c413ac56f09e9
- [ ] Create detection rules
- [ ] Build dashboards
- [ ] Simulate attacks
- [ ] Map detections to MITRE ATT&CK

<<<<<<< HEAD
## Repository Structure

```
architecture/     diagram source and exported image
docs/             setup guide, troubleshooting, and investigation writeups
screenshots/      raw evidence referenced from docs
detections/       SPL queries once built
=======
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
>>>>>>> ced275fa66f7c8a99a180a8e364c413ac56f09e9
```

## Roadmap

### Phase 1
- Windows Event Log collection
- Sysmon deployment
<<<<<<< HEAD
- Splunk Enterprise integration
=======
- Splunk Cloud integration
>>>>>>> ced275fa66f7c8a99a180a8e364c413ac56f09e9

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
<<<<<<< HEAD
- Multi endpoint monitoring
=======
- Multi-endpoint monitoring
>>>>>>> ced275fa66f7c8a99a180a8e364c413ac56f09e9
- Threat hunting scenarios
