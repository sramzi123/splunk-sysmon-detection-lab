# Troubleshooting: Sourcetype Rename Conflict Between Two Sysmon TAs

## Symptom

After installing and configuring `TA-microsoft-sysmon` on the Universal Forwarder — with an explicit `sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` override in `local/inputs.conf` — Sysmon events were still landing in Splunk tagged with the generic sourcetype `xmlwineventlog` instead of the expected CIM-mapped sourcetype.

Confirmed via `btool` that the forwarder-side config was correct:

```powershell
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" btool inputs list "WinEventLog://Microsoft-Windows-Sysmon/Operational" --debug
```

Output confirmed `sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` was correctly set and active. So the forwarder was tagging events correctly at collection time — but something downstream was overriding it.

## Root Cause

The local Splunk Enterprise indexer had a **second, separate add-on installed** — `Splunk_TA_microsoft_sysmon` (distinct from `TA-microsoft-sysmon` on the forwarder) — with its own `props.conf` containing an explicit rename rule:

```powershell
& "C:\Program Files\Splunk\bin\splunk.exe" btool props list "XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" --debug
```

```
C:\Program Files\Splunk\etc\apps\Splunk_TA_microsoft_sysmon\default\props.conf   rename = xmlwineventlog
```

This rule silently renamed any event tagged `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` back to the generic `xmlwineventlog` **at index time** — overriding whatever sourcetype the forwarder had correctly assigned. This is a classic multi-TA conflict: two add-ons intended for the same data source, each asserting different sourcetype normalization rules, with no inherent coordination between them.

## Fix

Created a local override on the indexer to neutralize the conflicting rename:

```powershell
New-Item -ItemType Directory -Path "C:\Program Files\Splunk\etc\apps\Splunk_TA_microsoft_sysmon\local" -Force
```

`C:\Program Files\Splunk\etc\apps\Splunk_TA_microsoft_sysmon\local\props.conf`:

```ini
[XmlWinEventLog:Microsoft-Windows-Sysmon/Operational]
rename =
```

Restarted the Splunk Enterprise instance to apply:

```powershell
& "C:\Program Files\Splunk\bin\splunk.exe" restart
```

Confirmed the fix with `btool` (showing the local override as the active value):

```powershell
& "C:\Program Files\Splunk\bin\splunk.exe" btool props list "XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" --debug | Select-String "rename"
```

```
C:\Program Files\Splunk\etc\apps\Splunk_TA_microsoft_sysmon\local\props.conf   rename =
```

Verified new events landing under the correct sourcetype with fully populated CIM fields:

```spl
index=main host="DESKTOP-HJSIADG" EventID=1 earliest=-2m
| stats count by sourcetype
```

## Lesson

Config precedence in Splunk isn't always intuitive when multiple add-ons touch the same data source — `btool` is the definitive way to see which file is actually "winning" for a given stanza, rather than guessing based on what should theoretically be configured. This kind of conflict (two overlapping TAs silently fighting over sourcetype normalization) is a realistic scenario in any environment where add-ons accumulate over time without a single owner reviewing overlaps.
