# Setup Walkthrough

This document covers how the lab was built: Sysmon installation, Universal Forwarder configuration, and getting Sysmon events properly parsed into Splunk's CIM (Common Information Model).

## Environment

- **Host:** Windows machine (`DESKTOP-HJSIADG`)
- **Splunk Enterprise:** installed locally, acting as indexer/search head
- **Splunk Universal Forwarder:** installed on the same host, collecting and forwarding logs
- **Index:** `main`

## 1. Install Sysmon

Downloaded Sysmon from Microsoft Sysinternals:
https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

Used the [SwiftOnSecurity Sysmon config](https://github.com/SwiftOnSecurity/sysmon-config), the industry-standard starting configuration for balanced signal-to-noise logging (avoids both default over-logging and under-logging).

Installed with:

```powershell
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

Verified the service was running:

```powershell
Get-Service sysmon64
```

Confirmed events were generating:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

## 2. Configure the Universal Forwarder to collect Sysmon logs

Installed the **TA-microsoft-sysmon** add-on (forwarder-side input and sourcetype tagging) into:

```
C:\Program Files\SplunkUniversalForwarder\etc\apps\
```

By default, the add-on ships with the Sysmon input **disabled**. To enable it without editing files under `default\` (which get overwritten on upgrades), created a `local` override:

```
C:\Program Files\SplunkUniversalForwarder\etc\apps\TA-microsoft-sysmon\local\inputs.conf
```

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = false
renderXml = 1
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

Restarted the forwarder service to apply the change:

```powershell
Restart-Service SplunkForwarder
```

## 3. Verify data is landing in Splunk

```spl
index=main host="DESKTOP-HJSIADG" sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| table _time, process_name, parent_process_name, dest, user, CommandLine
```

At this stage, `process_name` and `parent_process_name` should show real values (e.g. `cmd.exe`, `splunkd.exe`) — confirming Sysmon events are being correctly parsed into CIM-compliant fields, not just landing as raw XML.

## 4. Confirming configuration with `btool`

Splunk's `btool` is useful for confirming exactly which config file is "winning" for a given stanza, across all the layered `default`/`local` configs:

```powershell
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" btool inputs list "WinEventLog://Microsoft-Windows-Sysmon/Operational" --debug
```

This shows every file contributing settings to that input, in priority order — essential for catching config conflicts (see [troubleshooting.md](troubleshooting.md) for a real example of this in action).

## Result

A working pipeline: Sysmon → Windows Event Log → Universal Forwarder → Splunk indexer → CIM-mapped searchable fields (`process_name`, `parent_process_name`, `dest`, `user`, `CommandLine`, etc.), ready for detection engineering.
