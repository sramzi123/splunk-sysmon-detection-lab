# Home SOC Lab: Windows Event Logs + Sysmon in Splunk

A self-built security monitoring lab simulating a small-scale SOC data pipeline: Windows endpoint telemetry (Event Logs + Sysmon) forwarded into Splunk, parsed into CIM-compliant fields, and ready for detection engineering.

## Overview

This lab ingests native Windows Security/System/Application logs and Sysmon process, network, and file activity into Splunk, using the same architecture pattern (Universal Forwarder → Indexer, with Technology Add-ons for parsing) found in production SOC environments.

**Goal:** build hands-on experience with the full data pipeline a SOC analyst works with daily — not just running Splunk searches, but understanding how data gets from an endpoint into a usable, field-extracted format.

## Architecture

```mermaid
graph TD
    subgraph Host["DESKTOP-HJSIADG — single Windows host"]
        A[Sysmon<br/>SwiftOnSecurity config] --> B[Windows Event Log<br/>Security / System / Application / Sysmon-Operational]
        B --> C[Splunk Universal Forwarder<br/>TA-microsoft-sysmon]
        C --> D[Splunk Enterprise<br/>Indexer + Search Head<br/>Splunk_TA_microsoft_sysmon]
    end
    D --> E[(index=main)]
    E --> F[SPL Searches / Detections / Dashboards]
```

**Note:** the forwarder and indexer both run on the same physical host in this lab, rather than separate machines as in a typical production deployment. This surfaced a real config conflict — two independently-installed Sysmon add-ons (one on the forwarder, one on the indexer) both touching sourcetype normalization for the same data. See [`docs/troubleshooting.md`](docs/troubleshooting.md) for how that was diagnosed and fixed.

## What's in this repo

| Path | Description |
|---|---|
| [`docs/setup.md`](docs/setup.md) | Full setup walkthrough: Sysmon, Universal Forwarder, TA installation, and configuration |
| [`docs/troubleshooting.md`](docs/troubleshooting.md) | A real config conflict I diagnosed and fixed — two overlapping Sysmon TAs silently renaming sourcetypes at index time |

*(Detections, Atomic Red Team results, and dashboard screenshots to be added as the lab grows.)*

## Tech Stack

- Splunk Enterprise (local instance)
- Splunk Universal Forwarder
- Sysmon (SwiftOnSecurity configuration)
- TA-microsoft-sysmon (forwarder-side input/sourcetype)
- Splunk_TA_microsoft_sysmon (indexer-side CIM field extraction)

## Why this project

Built as part of my hands-on preparation for SOC Analyst / Security Analyst roles, following CompTIA Security+ certification. The focus here is understanding log pipeline architecture, sourcetype/CIM mapping, and the kind of data plumbing issues that come up in real environments — not just running canned searches.
