# Troubleshooting: The Case of the Sourcetype That Would Not Stick

## The problem

After getting Sysmon installed and the TA on my Universal Forwarder configured with an explicit sourcetype override, I expected events to land in Splunk tagged as XmlWinEventLog:Microsoft Windows Sysmon/Operational. Instead, every single event kept showing up as the generic xmlwineventlog sourcetype, no matter what I changed.

I checked the forwarder config with btool and it confirmed my sourcetype setting was correct and active.

```powershell
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" btool inputs list "WinEventLog://Microsoft Windows Sysmon/Operational" --debug
```

The output showed my sourcetype line sitting there, clearly winning. So the forwarder was doing exactly what I told it to do. Which meant the problem had to be happening somewhere after the forwarder, not because of it.

## Finding the actual cause

Since I also run a local Splunk Enterprise instance on the same machine, I checked the props configuration on the indexer side using the same tool.

```powershell
& "C:\Program Files\Splunk\bin\splunk.exe" btool props list "XmlWinEventLog:Microsoft Windows Sysmon/Operational" --debug
```

And there it was:

```
C:\Program Files\Splunk\etc\apps\Splunk_TA_microsoft_sysmon\default\props.conf   rename = xmlwineventlog
```

A completely separate add on, installed on the indexer and distinct from the one I had installed on the forwarder, had its own rule silently renaming any Sysmon event back down to the generic sourcetype the moment it hit the index. My forwarder was tagging things correctly the whole time. The indexer was just quietly undoing it.

This is a real trap. Two add ons with almost identical names, each doing something reasonable on its own, ended up fighting over the same data without either one knowing the other existed.

## The fix

I could not safely edit the file under default directly, since that gets overwritten whenever the add on updates. So I created a local override instead, in the proper spot for a config change like this.

```powershell
New-Item -ItemType Directory -Path "C:\Program Files\Splunk\etc\apps\Splunk_TA_microsoft_sysmon\local" -Force
```

Then wrote a blank rename value into a new props.conf there.

```ini
[XmlWinEventLog:Microsoft Windows Sysmon/Operational]
rename =
```

Restarted Splunk Enterprise to apply it.

```powershell
& "C:\Program Files\Splunk\bin\splunk.exe" restart
```

Confirmed the override had actually taken over using btool again.

```powershell
& "C:\Program Files\Splunk\bin\splunk.exe" btool props list "XmlWinEventLog:Microsoft Windows Sysmon/Operational" --debug | Select-String "rename"
```

And then checked fresh events coming in.

```spl
index=main host="DESKTOP-HJSIADG" EventID=1 earliest=-2m
| stats count by sourcetype
```

New events finally landed under the correct sourcetype, with fields like process_name and parent_process_name populated the way they were supposed to be all along.

## What I actually learned from this

Config precedence in Splunk is not something you can guess your way through. btool exists for a reason, and it turned out to be the only tool that could tell me, with certainty, which file was actually winning for a given setting. Assuming a config is correct because you wrote it that way is not the same as confirming it is the one actually being used.

The bigger lesson is about how real environments end up this way. Nobody sets out to install two conflicting add ons on purpose. They accumulate over time, installed by different people for reasonable reasons, and nobody checks whether they overlap until something stops making sense. That is exactly the kind of quiet, unglamorous problem a SOC analyst or Splunk admin runs into constantly, and it is a much better story to be able to tell than simply saying the install went smoothly.
